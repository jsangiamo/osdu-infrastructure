# OSDU R3-AKS Infrastructure Deployment Instructions

This section containts the steps required to deploy the OSDU infrastructure into Azure. Before completing this section, you should have completed everything under "What You Will Need". This infrastructure consists of three resource groups: common_resources (cr), service_resources (sr), and data_resources (dr).

## Deploy Infrastructure using Azure Dev Ops Pipelines
Follow the directions in [here](./deploy-infrastructure-using-pipelines.md).

## Manually Deploy Infrastructure

You will need to have several keys configured on your local machine when using the Terraform scripts locally:

```
# get node public key and gitops private key
az keyvault secret show --vault-name $SSH_VAULT_NAME -n "node-ssh-key-pub" --query value -otsv > ~/.ssh/node-ssh-key.pub
az keyvault secret show --vault-name $SSH_VAULT_NAME -n "gitops-ssh-key" --query value -otsv > ~/.ssh/gitops-ssh-key
# configure appropriate file permissions on these keys
chmod 644 ~/.ssh/node-ssh-key.pub
chmod 600 ~/.ssh/gitops-ssh-key
```
Update your `.envrc` file with the paths to your public and private SSH keys for Node and GitOPS repo access.

```
TF_VAR_ssh_public_key_file=/home/$USER/.ssh/node-ssh-key.pub
TF_VAR_gitops_ssh_key_file=/home/$USER/.ssh/gitops-ssh-key
```
You are now ready to run the three Terraform deployment scripts for OSDU on Azure. You can find the instructions for the common resources, data_resources, and service_resources in their respective READMEs:

Follow the directions in the [`common_resources`](../configurations/common_resources/README.md) environment.

Follow the directions in the [`data_resources`](../configurations/data_resources/README.md) environment.

Follow the directions in the [`service_resources`](../configurations/service_resources/README.md) environment.