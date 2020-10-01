# Azure OSDU AKS Service Deployment Instructions

## Interact with the Deployed Cluster

After `terraform apply` finishes for the service_resources, there is one critical output artifact: the Kubernetes config file for the deployed cluster that is generated and saved in the output directory. The default file is output/bedrock_kube_config. This file will let you interface with your deployed cluster. You can either declare this file explictly when using kubectl commands by adding `--kubeconfig output/bedrock_kube_config` (ex: `kubectl --kubeconfig output/bedrock_kube_config get pods --all-namespaces`), or you can follow [these](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/#append-home-kube-config-to-your-kubeconfig-environment-variable) instructions to configure kubectl to access your config file by default (you will need to `cp output/bedrock_kube_config ~/.kube/config`).

Here are some useful commands to interact with the deployed cluster:
* See all pods: kubectl get pods --all-namespaces
* Get osdu pods once services are deployed: kubectl get pods --namespace=osdu
* Describe a pod: kubectl describe \<name of pod> --namespace=\<namespace of pod>
* Get logs from a pod: kubectl logs -f \<name of pod> --namespace=\<namespace of pod>

## Continuous Deployment

Flux automation makes it easy to upgrade services or infrastructure. In this example Flux watches the repo we set up previously. Now we add a simple Web application to the running deployment by pushing a .yaml manifest to the repo. The .yaml specification describes the default-service and a Deployment. It specifies the source the Docker image that contains it: image: neilpeterson/aks-helloworld:v1 and how many containers to run: replicas: 1. 

When the .yaml file is complete we will push it to the repo, or simply drop it on GitHub. Flux is querying the repo for changes and will deploy the new service replicas as defined by this manifest.

Create the following .yaml file and name it default-service.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: default-service
  namespace: osdu
  labels:
    app: default-service
spec:
  type: ClusterIP
  ports:
    - port: 80
  selector:
    app: default-service
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: default-service
  namespace: osdu
spec:
  selector:
    matchLabels:
      app: default-service
  replicas: 1
  template:
    metadata:
      labels:
        app: default-service
    spec:
      containers:
        - name: default-service
          image: neilpeterson/aks-helloworld:v1
          ports:
            - containerPort: 80
          env:
            - name: TITLE
              value: "Azure OSDU Platform - (AKS)"
```

To see the changes as Flux picks them up and deploys them, you can monitor the logs from the flux container.

```
$ kubectl logs -f  flux-6899458bb8-qghrq --namespace=flux
```

Now, push or drop the default-service.yaml file to the empty repo created earlier in the infrastructure instructions. You can click `Upload files` on the GitHub repo page and drop the .yaml file:

Flux has connected to the repo and created the new service and deployment: `"kubectl apply -f -" took=1.591333771s err=null output="service/default-service created\ndeployment.apps/default-service created"`.

Now you can view the service deployed in the `osdu` namespace.

```bash
$ kubectl get services --namespace=osdu

NAME              TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
default-service   ClusterIP   10.0.212.75   <none>        80/TCP    117s


$ kubectl get deployments --namespace=osdu

NAME              READY   UP-TO-DATE   AVAILABLE   AGE
default-service   1/1     1            1           2m11s


$ kubectl get pods --namespace=osdu

NAME                               READY   STATUS    RESTARTS   AGE
default-service-86cd47b748-sc7bv   1/1     Running   0          2m19s
```

Finally we can connect directly to the pod and validate the service is properly running.

```bash
$ kubectl port-forward default-service-86cd47b748-sc7bv 8080:80

Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```

View the service in a browser http://localhost:8080

> To delete the service take the default-service.yaml and remove it from the manfiest repo and flux will perform the uninstall.


## Deploying OSDU Services
There are six OSDU services that can be deployed in this infrastructure: storage, search, indexer-serivce, indexer-queue, entitlements, and legal. Additionally, there an azure-common service that these other services depend on. The helm charts for these service can be found in their respective repos at /devops/azure:
* [Storage](https://community.opengroup.org/osdu/platform/system/storage/-/tree/master/devops/azure)
* [Search](https://community.opengroup.org/osdu/platform/system/search-service/-/tree/master/devops/azure)
* [Entitlements](https://community.opengroup.org/osdu/platform/security-and-compliance/entitlements-azure/-/tree/master/devops/azure)
* [Legal](https://community.opengroup.org/osdu/platform/security-and-compliance/legal/-/tree/master/devops/azure)
* [Indexer Service](https://community.opengroup.org/osdu/platform/system/indexer-queue/-/tree/master/devops/azure)
* [Indexer Queue](https://community.opengroup.org/osdu/platform/system/indexer-queue/-/tree/master/devops/azure)

The file that will be used to fill in the required variables for azure-common should be stored in the devops folder in the root of this repo. Here is a template for the file:
```yaml
# <infra-root>/devops/config.yaml
# This file contains the essential configs for the osdu on azure helm chart
global:

  # Service(s) Replica Count
  replicaCount: 2

  ################################################################################
  # Specify the azure environment specific values
  #
  azure:
    tenant: <tenantId>
    subscription: <subscriptionId>
    resourcegroup: <resourceGroupName> # name of your service resources resource group, will look like osdu-r3-aks-<name>sr-5dqv-rg
    identity: <osduIdentityName> # taken from inside service resources resource group, will look like <name>-osdu-r3-5dqv-aks-osdu-identity
    identity_id: <osduClientId> # taken from "Client ID" from Azure Portal for the osdu identity above, will look like 8f20299d-b8b2-4ffb-9706-...
    keyvault: <keyVaultName> # taken from inside service resources resource group, will look like  <name>-osdu-r3-5dqv-kv

  ################################################################################
  # Specify the Ingress Settings
  # DNS Hostname for thet Gateway
  # Admin Email Address to be notified for SSL expirations
  # Lets Encrypt SSL Server
  #     https://acme-staging-v02.api.letsencrypt.org/directory  --> Staging Server
  #     https://acme-v02.api.letsencrypt.org/directory --> Production Server
  #
  ingress:
    hostname: <hostname.domain.com> # taken from application gateway in service resources resource group, will look like <name>-osdu-r3-aks-jasonsansr-5dqv-gw.eastus.cloudapp.azure.com
    admin: <admin@email.com> # enter your email
    sslServer: https://acme-staging-v02.api.letsencrypt.org/directory  # Staging

  ################################################################################
  # Specify the Gitlab branch being used for image creation
  # ie: community.opengroup.org:5555/osdu/platform/system/storage/{{ .Values.global.branch }}/storage:latest
  #
  image:
    branch: master
```

To install the services using Flux, you will want a directory stucture like the following: 
```
project root
└───osdu-infrastructure (this repo)
└───osdu-services (folder you create)
|    └───entitlements-azure (service repo you will clone)
|    └───legal (service repo you will clone)
|    └───storage (service repo you will clone)
|    └───indexer-service (service repo you will clone)
|    └───indexer-queue (service repo you will clone)
|    └───search-service (service repo you will clone)
└───flux-repo (the repo for flux manifests you created earlier)
```
You can then use this script to generate the manifests in your flux repo after filling out your config.yaml file:
```bash
SRC_DIR="<ROOT_PATH_TO_SOURCE>"      #  $HOME/source/osdu
FLUX_SRC="<FULL_PATH_TO_SOURCE>"     #  $SRC_DIR/flux-repo
INFRA_SRC="<FULL_PATH_TO_SOURCE>"    #  $SRC_DIR/osdu-infrastructure
SERVICES_DIR="<FULL_PATH_TO_SOURCE>" #  $SRC_DIR/osdu-services
BRANCH="master"
TAG="latest"

# Extract manifests from the common osdu chart.
helm template osdu-flux ${INFRA_SRC}/devops/charts/osdu-common -f ${INFRA_SRC}/devops/config.yaml > ${FLUX_SRC}/providers/azure/hld-registry/azure-common.yaml

# Extract manifests from each service chart.
for SERVICE in entitlements-azure legal storage indexer-service indexer-queue search-service ;
do
  helm template $SERVICE ${SERVICES_DIR}/$SERVICE/devops/azure/chart --set image.branch=$BRANCH --set image.tag=$TAG > ${FLUX_SRC}/providers/azure/hld-registry/$SERVICE.yaml
done
```
After running this script, verify that the manifests are in your flux repo in the hld-registry directory and that the azure-common manifest is there as well. After verifying that these manifests are all in place, you can deploy the service by pushing the flux repo with the new files.

## Kubernetes Portal Dashboard

Kubernetes includes a web dashboard that can be used for basic management operations. This dashboard lets you view basic health status and metrics for your applications, create and deploy services, and edit existing applications.

```bash
$ kubectl create clusterrolebinding kubernetes-dashboard --clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard

$ az aks browse --resource-group myResourceGroup --name myAKSCluster
The behavior of this command has been altered by the following extension: aks-preview
Merged "devint-aks-mgf9wjxt-osdu-r2-aks" as current context in /tmp/tmps6_a6amm
Proxy running on http://127.0.0.1:8001/
Press CTRL+C to close the tunnel...
```

![image](https://user-images.githubusercontent.com/7635865/74479484-d54d3080-4e74-11ea-8160-ec4e087597ae.png)

