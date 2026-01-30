

# Tofu/Terraspace Configuration

### 1. Create terraspace config directory config/terraform 
The config/terraform folder is for common configurations. All files in the folder get built and included with the deployed module. Common examples:
- Backend Config: configure which backend to use by default. 
- Provider Config: configure which provider to use by default.
- Terraform Tfvars: Tfvars you want always set.
- Locals: Locals you want always available.

#### Example Terraspace folder/file Structure:
```
├── app
│   ├── modules
│   │   └── apim
│   └── stacks
│       └── apim-dev
└── config
    └── envs
        └── TS_ENV.rb
    └── helpers
        └── azure_auth_helper.rb
    └── plugins
        └── azurerm.rb
    └── terraform
        └── backend.tf
        └── provider.tf
        └── terraform.tfvars
├── Terrafile
├── Gemfile
```  

### 2. Create the Azure terraform backend.

# Remote State Storage Setup

In Terraspace, managing the Remote State (where your infrastructure's "source of truth" is stored) is handled automatically through the Booter process. For Azure, this typically involves an Azure Storage Account and a Blob Container.

**Here is how you set it up so Terraspace manages the state for you.**
Open (or create) the file at config/terraform/backend.tf. Terraspace uses Ruby templates to dynamically name your storage resources based on your environment.

Configure resource_group_name more dynamically:

(config/terraform/backend.tf) 
```
<%
def resource_group_name
  if ENV['APP']
    expansion("#{ENV['APP']}-:ENV-:LOCATION")
  else
    expansion(":ENV-:LOCATION")
  end
end
%>

terraform {
  backend "azurerm" {
    resource_group_name = "<%= resource_group_name %>"
    storage_account_name = "<%= expansion('ts:APP_HASH:SUBSCRIPTION_HASH:LOCATION_HASH:ENV') %>"
    storage_account_name = "<%= expansion('ts-state-:sub_account') %>" # Dynamic naming
    container_name = "terraform-state"
    key = "<%= expansion(':PROJECT/:REGION/:ENV/:BUILD_DIR/terraform.tfstate') %>"
  }
}
```
You can now provide APP and the resource group will account for it. Example:
```
APP=app1 terraspace up demo
```
### 3. Customize the Naming Logic
Because Azure Storage Account names must be globally unique and between 3-24 characters, you may want to customize the naming in (config/plugins/azure.rb):
```
Ruby
# config/plugins/azure.rb
TerraspacePluginAzure.configure do |config|
  config.storage_account.sku.name = "Standard_LRS"
  config.storage_account.location = "Australia East"
end
```

### 4. Configure terraspace itself with config/app.rb. 
This file is directly within the config folder.
Terraspace needs to know it should execute tofu instead of terraform. Configure this in your project so the AI sees it in your codebase and follows suit.

config/app.rb
```
Terraspace.configure do |config|
  config.logger.level = :info
  config.test_framework = "rspec"
  config.build.type = "opentofu" # Forces Terraspace to use tofu binary
end

```
#### Custom Logger
The default logger logs to $stderr. To use a custom logger to log to $stdout.
```
Terraspace.configure do |config|
  logger = Logger.new($stdout)
  logger.level = :debug
  config.logger = logger
end
```

### 5. Create the Opentofu azurem provider.tf file

(config/terraform/provider.tf)
- OpenTofu 4.x is now the standard for Azure. Ensure the global provider file) looks like this:

```
Terraform
terraform {
  required_providers {
    azurerm = {
      source  = "opentofu/azurerm"
      version = "~> 4.0"
    }
  }
}

provider "azurerm" {
  features {}
}
```
### 6. Create envirinments specific value overrides for config/app.rb with corresponding config/envs/TS_ENV.rb files.

config/envs/dev.rb
```
Terraspace.configure do |config|
  config.logger.level = :debug
end
```
config/envs/prod.rb
```
Terraspace.configure do |config|
  config.logger.level = :info
end
```
### 7. Generate a terraspace.rb helper 

Generating a check that automatically verifies your Azure authentication before running any terraspace commands

-  will save you from those annoying "Authentication Failed" errors midway through a deployment.

- In Terraspace, the best way to handle this is by creating a helper in the config/helpers directory. 
- This script will run before the Terraspace core logic kicks in, checking for an active Azure CLI session or service principal credentials.

**The Solution:** azure_auth_helper.rb
- You can create a file at config/helpers/azure_auth_helper.rb with the following logic:

```
module Terraspace::Helpers
  def verify_azure_auth
    puts "Checking Azure authentication..."
    
    # Check if the user is logged in via Azure CLI
    auth_check = `az account show --output json 2>&1`
    
    if $?.success?
      account_info = JSON.parse(auth_check)
      puts "Authenticated as: #{account_info['user']['name']}"
    else
      puts "❌ Error: Not authenticated with Azure."
      puts "Please run 'az login' before proceeding."
      exit 1
    end
  end
end

```

 It compares your active Azure CLI subscription against a mapping of your Terraspace environments.

Updated Helper: azure_auth_helper.rb

### Key Azure Settings
You can fine-tune how Terraspace handles Azure (like storage account names for state) in: config/plugins/azure.rb


## 8. Initialize and Create Resources
When you run a deployment command, Terraspace will check if the Resource Group and Storage Account defined in your backend actually exist. If they don't, it will offer to create them for you.

Run the plan:
```
Bash
terraspace plan resource_group
The Prompt: Terraspace will detect the missing backend and ask: Storage account ts-state-12345 does not exist. Would you like to create it? (y/N)

Confirm: Type y.
```
## 9. Verify the State
**9.1**  Once the command finishes, you can verify your state is stored in the cloud:

**9.2**  Log in to the Azure Portal.

**9.3**  Navigate to Storage Accounts.

**9.4** Look for the account name generated by Terraspace (e.g., tsstate12345).

**9.5** Under Containers > terraform-state, you will see your .tfstate file organized by project and environment.

## ⚠️ Pro-Tip: Environment Isolation
Terraspace uses the :ENV variable in the backend key. This means if you run TS_ENV=prod terraspace up resource_group, it will automatically store that state in a different folder than your dev environment, preventing accidental overrides.


# Terraspace Layering Setup

Terraspace Layering is one of the framework's most powerful features. It allows you to write your infrastructure code once and automatically inject different values based on your environment (dev, prod), region, or even Azure subscription.

## 1. How Layering Works
Terraspace looks into the seed or tfvars directories and searches for files that match your current environment. It merges them in a specific order of priority.

## 2. Setting Up Variables
Let's use the Resource Group example. We want the location and tags to change depending on whether we are in dev or prod.

### Create the Base Variables
Create a file for values that apply to all environments: (app/stacks/resource_group/tfvars/base.tfvars)
```
Terraform
location = "Australia East"
tags = {
  Project = "apim-shared-platform"
}
```
### Create Environment-Specific Variables
Now, create a file specifically for your development environment: app/stacks/resource_group/tfvars/dev.tfvars
```
Terraform
location = "Australia East"
tags = {
  Project     = "apim-shared-platform"
  Environment = "Development"
  ManagedBy   = "Terraspace"
}
```
Now, create a file specifically for your production environment: app/stacks/resource_group/tfvars/prod.tfvars
```
Terraform
location = "Australia East"
tags = {
  Project     = "apim-shared-platform"
  Environment = "Production"
  ManagedBy   = "Terraspace"
}
```


## 3. Update the Source Code
Ensure your stack is set up to accept these variables. Update a(pp/stacks/resource_group/variables.tf):
```
Terraform
variable "location" {
  type = string
}

variable "tags" {
  type = map(string)
}
```
Then, use them in (app/stacks/resource_group/main.tf):
```
Terraform
resource "azurerm_resource_group" "example" {
  name     = "rg-${var.tags["Project"]}-${terraform.workspace}"
  location = var.location
  tags     = var.tags
}
```
## 4. Deploying Different Environments
Terraspace uses the TS_ENV environment variable to decide which layer to use.

To deploy to Dev (default):
```
Bash
terraspace up resource_group
# Uses base.tfvars
```

To deploy to Prod:
```
Bash
TS_ENV=prod terraspace up resource_group
# Uses base.tfvars + prod.tfvars (overriding location to West US)
```
## 💡 Why this is great
Dry Code: You don't have separate main.tf files for dev and prod.

Safety: You can't accidentally deploy dev settings to prod because the variables are isolated by the TS_ENV flag.

---

# Terraspace Hooks

Terraspace Hooks are incredibly useful for automating tasks that sit just outside the "pure" infrastructure code—like running a security scan on your plan or notifying a Slack channel when a deployment finishes.

**Here is how to set up both Before and After hooks.**

## 1. Defining Hooks
Hooks are defined in your project's configuration files. You can set them globally (for all stacks) or locally (for one specific stack).

### Global Hooks
To apply a hook to every stack in your project, edit: (config/hooks/terraspace.rb)

### Stack-Specific Hooks
To apply a hook only to your resource_group stack, create: (app/stacks/resource_group/config/hooks.rb)

## 2. Example: Security Scan (Before Hook)
A common use case is running a security tool like tfsec or checkov before you allow a deployment to proceed.
```
Ruby
# config/hooks/terraspace.rb
before("up", "plan", 
  execute: "tfsec .",
  exit_on_fail: true # Stops the deployment if security issues are found
)
```
## 3. Example: Slack Notification (After Hook)
You can trigger a script to notify your team once the infrastructure has been successfully changed.
```
Ruby
# config/hooks/terraspace.rb
after("up", 
  execute: "curl -X POST -H 'Content-type: application/json' --data '{\"text\":\"🚀 Infrastructure deployed successfully!\"}' https://hooks.slack.com/services/YOUR/WEBHOOK/URL",
  on_error: :skip # Don't try to notify if the 'up' command failed
)
```

## 4. Hook Execution Order
When you run terraspace up, the execution follows this lifecycle:

  1. Before Hook: Runs your custom script (e.g., Linting/Security).
  2. Terraspace Build: Generates the OpenTofu/Terraform files.
  3. Terraform/OpenTofu Command: Runs the actual plan or apply.
  4. After Hook: Runs your post-deployment script (e.g., Cleanup/Notification).

## 🛠️ Advanced: Using Ruby for Hooks
Because the config files are Ruby, you can use logic to determine when a hook runs:
```
Ruby
after("up") do
  if ENV['TS_ENV'] == 'prod'
    puts "Production deployment complete. Sending logs to Datadog..."
    # Add logic here
  end
end
```

---
