# HCP Terraform Demo 
This repo includes some resources to showcase some of the features of HCP Terraform.

## Prerequisites

- Terraform (v1.9.8)
- An HCP Terraform account. The free tier account is enough for this demo ***except for no-code modules***. If you want to try no-code make sure to use a `plus` tier organization.
- Access to an Azure account for deploying resources. The cost of deploying the resources for the Hashicat app should amount to less than $20/mo, which means just a few cents for a quick demo.
- Access to the [GitHub demo org](https://github.com/tf-demos) with the predefined modules. Reach out to me for access.
- A Github account and a personal access token with the following permissions allowed:
    - repo:*
    - workflow
    - write:packages
    - delete: packages
    - admin:repo_hook
    - admin:org_hook
    - notifications
    This will be used for creating the modules within your organization from predefined modules.

## 1. Clone this repo and change to the new dir
```bash
git clone https://github.com/ricardorompar/hcp-terraform-demo-azure && cd hcp-terraform-demo-azure
```

## 2. Create an Azure service principal
In order to create these resources you'll need a Service Principal in your Azure subscription. You can create it using the Azure CLI (you can install it with `brew update && brew install azure-cli` on your Mac).

First, log in to your Azure account in the CLI:
```bash
az login
```

Create a Service Principal with `Contributor` role:
```bash
az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/<SUBSCRIPTION_ID>"
```

After running this command you'll get a response like the one below:

```json
{
  "appId": "9ce786...",
  "displayName": "azure-cli...",
  "password": "ZCP8...",
  "tenant": "237fb..."
}
``` 

## 3. Configure your variables

### Variables for Terraform config
Go over to `config-hcp-terraform` and create a `terraform.tfvars` with the following variables:

- `token`: a valid HCP Terraform token. Create one [here](https://app.terraform.io/app/settings/tokens)
- `email`: email address of the HCP Terraform account owner.
- `github-token`: a personal access token from GitHub with the permissions specified in the `Prerequisites` section.
- `arm_client_id`: the ID from `appId` obtained from the Service Principal creation above.
- `arm_client_secret`:the value from `password` obtained from the Service Principal creation above.
- `arm_subscription_id`: your Azure subscription ID.
- `arm_tenant_id`: your Azure tenant ID.

> Check the [example](./config-hcp-terraform/terraform.tfvars.example)file.

### Variables for modules deployment
Go over to `modules-demo` and create a `terraform.tfvars` with this variable:

- `prefix`: a memorable prefix for most of the resources' names.

> Check the [example](./modules-demo/terraform.tfvars.example)file.

### (Optional) Variables for no-code module
Go over to `no-code-module-demo` and create a `terraform.tfvars` with these variables:

- `token`: a valid HCP Terraform token. Create one [here](https://app.terraform.io/app/settings/tokens)
- `email`: email address of the HCP Terraform account owner.
- `github-token`: a personal access token from GitHub with the permissions specified in the `Prerequisites` section.

> Check the [example](./no-code-module-demo/terraform.tfvars.example)file.

## 4. Run the demo

This demo creates a variable set with the credentials needed to log in to your Azure account. This variable set will later be used to deploy all the required resources for the Hashicat app.

For creating the modules in the private registry you will need these modules in GitHub like [this](https://github.com/tf-demos). Reach out to me to request access to the org.

Go back to the root of the `hcp-terraform-demo-azure` directory and run these commands:

```bash
# Configure HCP Terraform: create a demo organization, workspace and modules
cd config-hcp-terraform
terraform init
terraform apply -auto-approve
cd .. #return

# Deploy the modules and infrastructure based on the configuration created before
cd modules-demo
terraform init
terraform apply -auto-approve
cd .. #return
```

### Optional: create a no-code module
This part of the demo is created separately because it requires a `plus` tier organization in HCP Terraform.


## 5. Cleanup