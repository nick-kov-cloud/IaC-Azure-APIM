Run a Smoke Test. This process will verify that your Azure CLI is authenticated, Terraspace is correctly configured with the OpenTofu engine, and your Azure Backend (Storage Account) can be automatically provisioned.

### 1. Create a "Smoke Test" Stack
We’ll create a minimal stack that just deploys a Resource Group. This avoids the complexity of APIM while we verify the plumbing.
```
Bash
# Create the stack
terraspace new stack smoke_test
```
2. Define the Resources
Edit app/stacks/smoke_test/main.tf:
```
Terraform
resource "azurerm_resource_group" "test" {
  name     = "rg-smoke-test-${var.env}"
  location = "East US"
  
  tags = {
    ManagedBy = "Terraspace"
    Engine    = "OpenTofu"
  }
}
```
Edit app/stacks/smoke_test/variables.tf:
```
Terraform
variable "env" {
  type    = string
  default = "dev"
}
```
### 3. The Smoke Test Execution
This is where the magic happens. Run the following command:
```
Bash
terraspace up smoke_test
```
What to watch for:
Dependency Check: Terraspace will check for tofu (OpenTofu) and az.

Backend Initialization: If this is your first time, Terraspace will notice the Azure Storage Account for your remote state doesn't exist yet.

Action: It will ask: Storage account tsstate... does not exist. Create it? (y/N).

Response: Type y.

Security Hook: You should see your tfsec hook run. Since a Resource Group is inherently safe, it should pass.

OpenTofu Plan: It will show you that 1 resource (the Resource Group) is about to be created.

Apply: Confirm with yes.

### 4. Verify in the Azure Portal
Once the command finishes successfully:

Check the State: Go to the Azure Portal -> Storage Accounts. You should see a new account containing your terraform.tfstate.

Check the Resource: Go to Resource Groups. You should see rg-smoke-test-dev.

### 5. Clean Up
To avoid unnecessary costs or "clutter" in your Azure subscription, destroy the test resources once verified:
```
Bash
terraspace down smoke_test
```

✅ Success!
If that worked, your entire IaC factory is operational. You are ready to deploy the complex APIM Standard V2 architecture we built earlier.