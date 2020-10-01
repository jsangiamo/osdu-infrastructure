# Azure OSDU R3 - How to Set Up Environment Variables

This page describes how to set up environment variables when deploying this infrastructure. We use [direnv](https://direnv.net/) to source our environment variables. You will create a .envrc file to store these variables: `touch <project_root>/infra/templates/osdu-r3-resources/.envrc`. This file will store the information Terraform needs to deploy the infrastructure and you will fill it out as you complete the instructions in this repo. Here is the template you will be using for this deployment:
```bash
# .envrc in <project_root>/infra/templates/osdu-r3-resources/

# Subscription Info (filled in before starting with the id of the subscription you want your resources in)
    export ARM_SUBSCRIPTION_ID="<you will fill this in>"

# Terraform Service Principal Info (filled in with info for sp you will be using)
# See https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-create-service-principal-portal for documentation on where these attributes are
    export ARM_TENANT_ID="<id of tenant the sp is in>"
    export ARM_CLIENT_ID="<client id of the sp>"
    export ARM_CLIENT_SECRET="<sp secret>"

# Keyvault Storing ssh Keys (created manually in prerequisite instructions)
    export SSH_VAULT_NAME="<you will fill this in>"

# Terraform State Storage Info (filled in when creating Azure resources to store Terraform state)
    export TF_VAR_resource_group_location="<resource group location with Terraform state resources>"
    export TF_VAR_remote_state_account="<name of the storage account you created>" 
    export TF_VAR_remote_state_container="<name of the container you created>" 
    export ARM_ACCESS_KEY="<account key for the storage account you created" # can be retrieved using something like :
    #ACCOUNT_KEY=$(az storage account keys list --resource-group $RESOURCE_GROUP_NAME --account-name $STORAGE_ACCOUNT_NAME --query [0].value -o tsv)

# SSH key locations (this is where the instructions store them when pulling from the ssh keyvault if followed)
    export TF_VAR_ssh_public_key_file=/home/$USER/.ssh/node-ssh-key.pub
    export TF_VAR_gitops_ssh_key_file=/home/$USER/.ssh/gitops-ssh-key

# Workspace names (the instructions set these if followed, only change if you deviate when deploying the common resources or data resources)
    export TF_VAR_common_resources_workspace_name="${USER}-cr" 
    export TF_VAR_data_resources_workspace_name="${USER}-dr" 

# Elastic Search Info (created before deploying using these instructions, must be version 6.8.x and have a valid ssl certificate for https)
    export TF_VAR_elasticsearch_endpoint="<you will fill this in>"
    export TF_VAR_elasticsearch_username="<you will fill this in>"
    export TF_VAR_elasticsearch_password="<you will fill this in>"

# Flux repo info (taken from the Flux repo you create)
    export TF_VAR_gitops_ssh_url="<you will fill this in>" # you can find this in the Flux repo you create, it will look something like git@github.com:Azure/osdu-infrastructure-flux.git
    export TF_VAR_gitops_branch="master" # only change if you want Flux to watch a branch other than master

# Cosmos replica info (you set this to what you want)
    export TF_VAR_cosmosdb_replica_location="centralus" # this is just a default value, change it if you'd like

# Configurations, do not touch
    export BUILD_BUILDID=1
    export GO_VERSION=1.12.5
    export TF_VERSION=0.12.4

```
Note: remember to "direnv allow" into a bash terminal whenever these values change so that they are properly updated.