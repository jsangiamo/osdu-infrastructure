# Azure OSDU AKS Architecture Solution with Elastic Cloud SaaS

The `osdu` - Kubernetes Architecture solution template is intended to provision Managed Kubernetes resources like AKS and other core OSDU cloud managed services like Cosmos, Blob Storage and Keyvault. 

We decided to split these configuration files out into a separate Terraform environment templates to mitigate the risk of Terraform accidentally deleting stateful resources types as well as have a mechanism to duplicate environments to support concepts such as data partitioning or multiple AKS Clusters.


## Technical Design
Technical design [specifications](docs/aks-environment.md)

## GitOps Design
GitOps design [specifications](../../../docs/osdu/GITOPS_DESIGN.md).

## Cloud Resource Architecture
<details>
  <summary>Click to see diagram</summary>

  ![Architecture](./docs/images/architecture.png "Architecture")

</details>

## Resource Topology

<details>
  <summary>Click to see diagram</summary>

![Resource Topology](./docs/images/topology.png "Resource Topology")

</details>

## Terraform Template Topology
<details>
  <summary>Click to see diagram</summary>

![Template Topology](./docs/images/template.png "Terraform Template Topology")

</details>

## Intended Audience

Cloud administrators who are versed with both Cobalt templating and Kubernetes.

## What You Will Need
1. Local environment variables [setup](docs/setup-environment-variables.md).
1. A target Azure subscription into which you will deploy. Take this subscription's id from within the Azure Portal and use it to fill `ARM_SUBSCRIPTION_ID` in your .envrc file.
1. A Service Principal that (1) has Azure Active Directory Graphy app role Application.ReadWrite.OwnedBy granted with admin consent and (2) is granted owner level role assignment in target subscription. You should then take this service principal information and use it to fill in the section under "Terraform Service Principal Info" in your .envrc file.
1. Azure Storage Account that is [setup](https://docs.microsoft.com/en-us/azure/terraform/terraform-backend) to store Terraform state (you only need to complete the steps under "Configure Storage Account"). You should take this information and fill in the section under "Terraform State Storage Info" in your .envrc file.
1. A keyvault that you create in the Azure Portal. You may choose to put this key vault into the resource group you created for your terraform state. You should take the name of this keyvault and use it to fill in `SSH_VAULT_NAME` in your .envrc file. 
1. A 6.8.x Elasticsearch instance with a valid ssl certificate for https. You can take the endpoint, username, and password for this instance use them to fill in `TF_VAR_elasticsearch_endpoint`, `TF_VAR_elasticsearch_username`, and `TF_VAR_elasticsearch_password` respectively in your .envrc file.
1. An Ubuntu terminal. Windows users can install the [Ubuntu Terminal](https://www.microsoft.com/store/productId/9NBLGGH4MSV6) from the Microsoft Store. The Ubuntu Terminal enables Linux command-line utilities, including bash, ssh, and git that will be useful for the following deployment. _Note: You will need the Windows Subsystem for Linux installed to use the Ubuntu Terminal on Windows_.
1. Software requirements:
    * Terraform (0.12.28 required)
    * Go version (1.12.14 required)
    * Helm
    * Kubectl
    * Azure CLI


## Cost
Azure environment cost ballpark [estimate](https://azure.microsoft.com/en-us/pricing/calculator/?shared-estimate=61d8d0ef1644470c9c23d6d51796b4b7). This is subject to change and is driven from the resource pricing tiers configured when the template is deployed.
## Deploying the OSDU Infrastructure
Once all of the prerequisites are completed, you can start the OSDU deployment process by deploying the Azure infrastructure on which the services will run. See instructions [here](docs/infrastructure-instructions.md)

## Deploying OSDU Services Into Infrastructure
Once your Azure infrastructure has been deployed, you can load the OSDU services onto that infrastructure. See instructions [here](docs/service-instructions.md)

## License
Copyright © Microsoft Corporation

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at 

[http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0)

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.