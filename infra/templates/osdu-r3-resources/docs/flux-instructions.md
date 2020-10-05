# OSDU R3-AKS Flux Setup Instructions
In this section you will create and configure the repository used by Flux. Flux is a tool that transforms the state of a kubernetes cluster to match the configuration specified in Git. The repo you will be creating in this section is the repo into which you will upload the manifests that define services in OSDU and will be used be Flux to deploy those services in Kubernetes.

## Flux Setup: Creating Flux Repository
In this section you will be creating an empty GitHub repository. Once your Kubernetes cluster is deployed later in this guide, it will be configured to watch this repository. As you push helm charts to this repository the service those charts represent will be deployed into your cluster.

If you are going to be using ADO to create CI/CD pipelines for your OSDU deployment, you can follow [these](https://docs.microsoft.com/en-us/azure/devops/repos/git/create-new-repo?view=azure-devops) instructions to create a new repo.

If you want to manually deploy OSDU without CI/CD, you can create an empty repo yourself on the [GitHub website](https://github.com/).

Either way, once you have created your repo Flux requires there be at least one commit. You can create an empty commit so Flux will work correctly with your empty repo:
`git commit --allow-empty -m "Initializing the Flux Manifest Repository"`
When you are deploying services, you will put the manifests in providers/azure/hld-registry directory of this repo. Once you push the manifests, Flux should automatically deploy the services.

After creating the Flux repository, you can take its ssh url from GitHub (it will look something like git@github.com:Azure/osdu-infrastructure-flux.git) and it to your .envrc file as `TF_VAR_gitops_ssh_url`.

## Flux Setup: Creating Gitops and Node Keys
In this section you will be creating two ssh keys that will be used when deploying the OSDU infrastructure. These keys will be stored in the keyvault you created earlier that has its name stored in the `SSH_VAULT_NAME` environment variable.

The first key you will be creating is the gitops key that will be used to deploy service charts that you put in your Flux repo into the Kubernetes cluster you will create:
```bash
KEY_NAME=gitops-ssh-key

# Generate gitops-ssh-key
ssh-keygen -b 4096 -t rsa -f $KEY_NAME

# Save gitops-ssh-key
az keyvault secret set --vault-name $SSH_VAULT_NAME -n "${KEY_NAME}" -f "${KEY_NAME}"
az keyvault secret set --vault-name $SSH_VAULT_NAME -n "${KEY_NAME}-pub" -f "${KEY_NAME}.pub"

# Delete keys from your local machine to be more secure
rm $KEY_NAME
rm $KEY_NAME.pub

# Show Public gitops-ssh-key
az keyvault secret show --vault-name $SSH_VAULT_NAME -n "${KEY_NAME}-pub" --query value -otsv
```
If you are using ADO for your deployment, follow these [steps](https://docs.microsoft.com/en-us/azure/devops/repos/git/use-ssh-keys-to-authenticate?view=azure-devops&tabs=current-page#step-2--add-the-public-key-to-azure-devops-servicestfs) to add your public SSH key to your ADO environment.

If you are doing a manual deployment without ADO, you now want to take the public key you just printed to the console and add it as a [deploy key](https://developer.github.com/v3/guides/managing-deploy-keys/#deploy-keys) to your Flux repo.

The second key you will be creating is node key that Terraform uses to setup log-in credentials on the nodes in the AKS cluster. We will use this key when setting up the Terraform deployment variables. Generate the Node Key:

```bash
KEY_NAME=node-ssh-key

# Generate node-ssh-key
ssh-keygen -b 4096 -t rsa -f $KEY_NAME

# Save node-ssh-key
az keyvault secret set --vault-name $SSH_VAULT_NAME -n "${KEY_NAME}" -f "${KEY_NAME}"
az keyvault secret set --vault-name $SSH_VAULT_NAME -n "${KEY_NAME}-pub" -f "${KEY_NAME}.pub"

# Delete keys from your local machine to be more secure
rm $KEY_NAME
rm $KEY_NAME.pub

# Save Locally Public node-ssh-key
az keyvault secret show --vault-name $SSH_VAULT_NAME -n "${KEY_NAME}-pub" --query value -otsv
```
Once both keys have stored in the vault, you are ready to proceed to the [next section](./infrastructure-instructions.md) in which you will be deploying the infrastructure on which OSDU will run.