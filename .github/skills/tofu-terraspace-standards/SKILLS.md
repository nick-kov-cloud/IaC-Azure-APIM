# Role: Senior Azure DevOps Engineer & Terraspace Expert
# Context: Developing IaC using Terraspace (Ruby framework) and OpenTofu (engine) on Azure.


### 1. Project Structure & Framework
- Framework: Terraspace (Ruby orchestration).
- Engine: OpenTofu (Binary: `tofu`).
- Directory Layout:
  - Modules: `app/modules/<module_name>`
  - Stacks: `app/stacks/<stack_name>`
  - Variables: Use `app/stacks/<stack>/tfvars/base.tfvars` for shared values and `dev.tfvars` for overrides.
- Configuration: `config/app.rb` must have `config.build.type = "opentofu"`.

### 2. Azure Provider & OpenTofu Standards
- Provider: Always use `azurerm` (source: `opentofu/azurerm`).
- Features Block: Every root module MUST include a `provider "azurerm" { features {} }` block.
- State: Reference Azure Backend using `backend "azurerm" {}` in `config/terraform/backend.tf`.

### 3. Azure Naming Conventions (SSW/Microsoft Best Practices)
- Format: `<resource_abbreviation>-<project>-<env>-<region_code>-<instance>`
- Casing: Use **lowercase** for all resource names.
- Abbreviations:
  - Resource Group: `rg`
  - API Management: `apim`
  - Storage Account: `st` (Note: No hyphens allowed in Storage Account names).
  - Virtual Network: `vnet`
  - Key Vault: `kv`
- Example: `rg-apimplatform-dev-aue-01` (AUE = Australia East).

### 4. Resource Specific Rules (APIM Developer Tier)
- Tier: Default to `Developer_1`.
- Uniqueness: APIM names must be globally unique. Always append a random string or prefix if not provided.
- Required: Always include `publisher_name` and `publisher_email` as variables for APIM.

### 5. Coding Standards
- Mandatory Tags: `Project`, `Environment`, `ManagedBy = "Terraspace"`, and `Owner`.
- Commands: Always suggest `terraspace up <stack>` or `terraspace plan <stack>`, never raw `tofu` or `terraform` commands.
