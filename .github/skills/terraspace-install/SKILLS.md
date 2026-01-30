# Installing Terraspace

Terraspace is a Ruby-based framework that acts as a wrapper for Terraform or OpenTofu. It provides a structured way to keep your infrastructure-as-code (IaC) dry and organized.

**Note:** As of 2026, Terraspace officially supports and recommends using OpenTofu as the underlying engine.

## 🛠️ Prerequisites
Before installing Terraspace, you must have the following installed:

- Terraform (v0.12+) or OpenTofu (Recommended).
- Ruby (v3.0 or higher is recommended).
- Cloud Provider CLI (e.g., AWS CLI, Azure CLI, or Google Cloud SDK).

## 🚀 Installation Options
Option 1: Using RubyGems (Recommended)
If you already have Ruby installed, this is the most flexible method.
```
Bash
gem install terraspace
```

## Option 2: Standalone Installer (Linux)
If you don't want to manage Ruby yourself, use the standalone installer which bundles everything into /opt/terraspace.

## For Ubuntu/Debian:
```
Bash
curl -s https://apt.boltops.com/boltops-key.public | sudo apt-key add -
echo "deb https://apt.boltops.com stable main" | sudo tee /etc/apt/sources.list.d/boltops.list
sudo apt-get update
sudo apt-get install -y terraspace
```
## For RHEL/CentOS/Fedora:

```
Bash
sudo curl -so /etc/yum.repos.d/boltops.repo https://yum.boltops.com/boltops.repo
sudo rpm --import https://yum.boltops.com/boltops-key.public
sudo yum install -y terraspace
```

## Option 3: macOS (Homebrew)
The standalone Mac installer is deprecated in favor of Homebrew.
```
Bash
brew tap boltops-tools/software
brew install terraspace
```

## 🏗️ Setup and Verification
After installation, verify it works and run the setup check to ensure all dependencies (like OpenTofu) are found.
```
Bash
terraspace -v
terraspace setup check
```

Create Your First Project
Once installed, you can generate a new project structure:
```
Bash
terraspace new project azure --plugin azurerm
cd azure
terraspace new stack resource_group
```

## 🐳 Docker (Alternative)
If you want to avoid local installation entirely, you can use the official Docker image:
```
Bash
docker pull boltops/terraspace
```

## ✅ Next Step: Terraspace for Azure


To set up Terraspace for Azure, you’ll need to configure the provider and authenticate your environment. Since you’ve already installed OpenTofu and Terraspace, here are the steps to get your Azure infrastructure rolling.

## 1. Configure the Azure Plugin
Terraspace uses plugins to handle provider-specific logic. If you haven't already, install the Azure plugin:
```
Bash
gem install terraspace_plugin_azure
```
## 2. Initialize the Project
If you are starting fresh, run the generator to create the directory structure tailored for Azure:
```
Bash
terraspace new project my-azure-infra --plugin azure
cd my-azure-infra
```
## 3. Azure Authentication
Terraspace relies on the standard Azure CLI or Service Principal environment variables. The easiest way to start is by logging in via the CLI:
```
Bash
az login
```
Alternatively, for CI/CD or Automation, set these environment variables:
```
ARM_CLIENT_ID

ARM_CLIENT_SECRET

ARM_SUBSCRIPTION_ID

ARM_TENANT_ID
```
## 4. Create a Resource Group Stack
Let's create a "stack" (a collection of resources) for a Resource Group.

Generate the stack:
```
Bash
terraspace new stack resource_group
Define the resource: Open app/stacks/resource_group/main.tf and add:
```

```
Terraform
resource "azurerm_resource_group" "example" {
  name     = "terraspace-rg"
  location = "Australia East"
}
```
## 5. Deploy
Deploying with Terraspace is different from standard OpenTofu; it builds the backend and provider configurations automatically before running the plan.

```
Bash
terraspace up resource_group
```
