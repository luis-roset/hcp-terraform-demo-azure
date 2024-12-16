# HCP Terraform Demo 
This repo includes some resources to showcase some of the features of HCP Terraform.

## Prerequisites

- Terraform (v1.9.8)
- An HCP Terraform account. The free tier account is enough for this demo ***except for no-code modules***. If you want to try no-code make sure to use a `plus` tier organization.
- Dev access to EntraID to create the service account. Request access here: https://docs.prod.secops.hashicorp.services/doormat/azure/working_with_ad/ 
- Access to an Azure account for deploying resources. The cost of deploying the resources for the Hashicat app should amount to less than $20/mo, which means just a few cents for a quick demo.
- A GitHub account and granted access to the [GitHub demo org](https://github.com/tf-demos) with the predefined modules. Reach out to me for access.
- A GitHub personal access token with the following permissions allowed:
    - org:read
    This will be used for creating the modules within your organization from predefined modules.

## 1. Clone this repo and change to the new dir
```bash
git clone https://github.com/ricardorompar/hcp-terraform-demo-azure && cd hcp-terraform-demo-azure
```

## 2. Create an Azure service principal
In order to create these resources you'll need a Service Principal in your Azure subscription. You can create it using the Azure CLI (you can install it with `brew update && brew install azure-cli` on your Mac).

First, log in to your Azure account in the CLI:
```bash
az login --tenant <TENANT_ID>
```

Create a Service Principal with `Contributor` role:
```bash
az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/<SUBSCRIPTION_ID>"
```

After running this command you'll get a response with a payload like the one below:

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

> Check the [example](./config-hcp-terraform/terraform.tfvars.example) file.

### Variables for modules deployment
Go over to `modules-demo` and create a `terraform.tfvars` with this variable:

- `prefix`: a memorable prefix for most of the resources' names.

> Check the [example](./modules-demo/terraform.tfvars.example) file.

### Variables for Vault cluster
Go over to `vault` and create a `terraform.tfvars` with these variable:

- `prefix`: a memorable prefix for most of the resources' names.
- `hcp_project_id`: the ID of the project in HCP to deploy the Vault cluster.

> Check the [example](./vault/terraform.tfvars.example) file.

## 4. Run the demo

This demo creates a variable set with the credentials needed to log in to your Azure account. This variable set will later be used to deploy all the required resources for the Hashicat app.

For creating the modules in the private registry you will need the predefined modules in GitHub like [this](https://github.com/tf-demos).

You may also create your own modules that you can clone from [these repositories](https://github.com/orgs/tf-demos/repositories) and change the `identifier` value in the modules definition in the [`modules.tf`](./config-hcp-terraform/modules.tf) file. Check [this guide](https://developer.hashicorp.com/terraform/cloud-docs/registry/publish-modules) to learn more about creating and publishing modules in your private registry.

### `New`: Create a Vault cluster and consume secrets from Vault
> ⚠️ Note: this requires an account in HCP. 
>
> ⚠️ The deployment of this (development) cluster in Azure takes around 10-15 minutes.

This will create a `dev` cluster by default. The outputs are a token with restricted policies to only read the example secret that's configured and the address of the cluster.

This Vault instance will be used to showcase the consumption of secrets from the new version of the Hashicat app.

```bash
cd vault
terraform init
terraform apply -auto-approve
cd .. #return
```

### Configure HCP Terraform and deploy resources.

With this demo you will deploy the Hashicat app with the resources shown in this diagram:

![Infrastructure_diagram](./src/diagram.png)

Go back to the root of the `hcp-terraform-demo-azure` directory and run the following commands.
```bash
# Configure HCP Terraform: create a demo organization in HCP Terraform, workspace and modules
cd config-hcp-terraform
terraform init
terraform apply -auto-approve
cd .. #return
```

With this configuration in place you can use the Designer configuration in HCP Terraform or deploy a no-code module.

### Optional: deploy resources with the CLI-driven workflow

> NOTE: This should take around 3-4 minutes to deploy.

```bash
# Deploy the modules and infrastructure based on the configuration created right before
cd modules-demo
terraform init
terraform apply -auto-approve
cd .. #return
```

## 5. Cleanup

Destroy the infrastructure created with the no-code workflow in HCP in case you deployed something.

In case you also created some infrastructure with the CLI workflow:
```bash
cd modules-demo
terraform destroy -auto-approve
cd .. #return
```
Destroy all configurations created for HCP Terraform:
```bash
# Destroy infra from modules-demo
cd config-hcp-terraform
terraform destroy -auto-approve
cd .. #return
```

Destroy Vault cluster and configs:
```bash
cd vault
terraform destroy -auto-approve
cd .. #return
```
