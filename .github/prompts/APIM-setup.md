
## Check Prerequisites

### Validate my local setup: 
  - check terraspace version
```
terraspace version
```
  - check tofu version
```
tofu version    
```
  - check azure cli
```
az version
```

### 1. Generate all base teraspace and tofu/terraform configuration files for a new project.
- refer to guide for initial terraspace and tofu setup: **[tofu-terraspace-setup](../skills/tofu-terraspace-setup/SKILLS.md)** 
- run the smoke-test: refer to **[smoke-test](../skills/smoke-test/SKILLS.md)
- Verify teraspace and tofu are working

### 2. Generate Project structure for APIM - terraspace and terraform


"Create a terraspace stack in app/stacks/apim for the creation of Azure APIM Gateway which includes a reusable tofu module.

Stack should create a new resource group if one is defined and does not exist. It should use a resource group tofu module and reference the module.

Apply the standard tags from our rules file.

Generate main.tf, variables.tf, locals.tf, outputs.tf and tfvars/base.tfvars."