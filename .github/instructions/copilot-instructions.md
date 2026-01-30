---
name: copilot
description: Core instructions for provisioning an Enterprise grade shared Azure APIM Service. Follow these guidelines for all Opentofu and Terraspace HCL best practices and security best practices.
---

## ⚠️ MANDATORY Development Checklist

**Before pushing ANY code, complete ALL of the following:**

- [ ] **Lint**: Run `terraspace validate` with zero warnings
- [ ] **Build**: Run `terraspace plan` with zero errors

**Failure to complete this checklist will result in work being rejected.**

---

## Project Overview

**APIM Shared Platform Provisioning** is a project to provision an Enterprise grade shared Azure API Management (APIM) service. This project uses Opentofu and Terraspace to define and manage the infrastructure as code (IaC).  

The goal is to create a scalable, secure, and maintainable APIM service that can be shared across multiple teams and applications within the organization. The project adheres to best practices in both Opentofu and Terraspace, ensuring that the infrastructure is robust and compliant with security standards.md template provided in the repository.


1.


---

## Quick Start Guide1. 
1. **Clone the Repository**: Start by cloning the repository to your local machine.
2. **Create a feature branch**: Create a new branch for your work to keep changes organized.
   ```Bash
   git checkout -b feature/your-feature-name
3. **Run Initial Setup**: Execute any initial setup and Install Dependencies if required
  - Navigate to the project directory and run all necessary dependencies:  
  - **[opentofu](../skills/opentofu-install/SKILLS.md)**
  - **[terraspace](../skills/terraspace-install/SKILLS.md)**  
  - **[azure-cli](../skills/azure-cli-install/SKILLS.md)**
4. **Authenticate with Azure**: Use `az login` to authenticate your Azure CLI with your Azure account.
5. **Set Subscription**: If you have multiple subscriptions, set the active subscription using:  
   ```Bash
   az account set --subscription "YOUR_SUBSCRIPTION_ID"
   ```

6. **Project & Provider Standards**
- Engine: OpenTofu (Binary: `tofu`).
- Framework: Terraspace (Orchestration).
- Provider: `azurerm` (Source: `opentofu/azurerm`, Version: `~> 4.0`).
- Global Config: Every stack must include `provider "azurerm" { features {} }`.
- Terraspace Settings: Ensure `config.build.type = "opentofu"` in `config/app.rb`.
- Terraspace needs to know it should execute tofu instead of terraform. You should configure this in your project so the following is set:
- Update config/app.rb:
```
Terraspace.configure do |config|
  config.build.type = "opentofu" # Forces Terraspace to use tofu binary
end
```
6. **Generate Terraspace Config folders and files**: Run the terraspace config generation steps as per the project guidelines:  
  - **[terraspace config](../skills/tofu-terraspace-setup/SKILLS.md)**
4. **Configure Backend**: Ensure that the Terraspace backend is configured to use Azure Storage for remote state management.
5. **Set Environment Variables**: Configure any required environment variables for authentication and configuration.
7. **Run a smoke test**: The smoke test will generate a stack for provisioning resource groups and verify that everything is working correctly:
  - **[Run a smoke test](../skills/smoke-test/SKILLS.md)**


## APIM Service Provisioning Steps

1. **Create Resource Group tofu module**: Create a new tofu reusable module.
2. **Create APIM reusable tofu module**: Create a new tofu reusable module for APIM.
  Reference: - **[APIM Design Guide](../../docs/APIM-V2-TIER.md)**
3. **Create Terraspace Stack for APIM**: Generate a new corresponding Terraspace stack for the APIM module.
    - utilise the resource group module to create a new resource group and APIM service.
    - utilise best practices for naming conventions, tagging, and configuration as per the project guidelines.
      - **[Enterprise Best Practices](../../docs/ENTERPRISE-BEST-PRACTICES.md)**
    - utilise security best practices for APIM deployment.
      - **[Security Best Practices](../../docs/SECURITY-BEST-PRACTICES.md)**
4. **Define APIM Resources**: In the newly created stack, define the necessary resources for the APIM service in the `main.tf`, `variables.tf`, and `outputs.tf` files.
5. **Initialize Terraspace**: Run `terraspace init` to initialize the Terraspace project.
6. **Validate Configuration**: Execute `terraspace validate` to ensure the configuration is correct.
7. **Plan Deployment**: Use `terraspace plan` to see the changes that will be applied.
8. **Deploy Initial Resources**: Use `terraspace up` to deploy the initial resources and verify the deployment in the Azure portal. 
9. **Verify Deployment**: Check the Azure portal to verify that the APIM service has been provisioned correctly.
