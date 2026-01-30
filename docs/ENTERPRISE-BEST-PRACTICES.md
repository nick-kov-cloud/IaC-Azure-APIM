
### OpenTofu & Terraspace
- **State Isolation**: Use separate state files for different environments (dev, staging, production).
- **Remote State**: Configure remote state backend in Azure Storage to enable team collaboration.
- **Planning & Review**: Always run `tofu plan` and review changes before applying with `tofu apply`.
- **Modularity**: Organize code into reusable modules for maintainability and scalability.

### Azure APIM Specific
- **API Versioning**: Plan your API versioning strategy before deployment.
- **Policies**: Define API policies (throttling, authentication, caching) consistently.
- **Monitoring**: Enable Application Insights and configure diagnostic settings for APIM.
- **SKU Selection**: Choose appropriate SKU based on throughput and feature requirements.

### Deployment
- **Automation**: Implement CI/CD pipelines to automate plan and apply stages.
- **Environment Parity**: Keep development, staging, and production configurations as similar as possible.
- **Rollback Strategy**: Plan and document rollback procedures for infrastructure changes.

# Reusable Opentofu Modules

To make your code reusable, we'll create the core APIM logic into a local module. This allows you to call the same APIM logic for different environments (Dev, QA, Prod) while keeping the configuration isolated in your stacks.

### 1. Create the Reusable Module
In Terraspace, local modules live in the vendor/modules or app/modules directory. 
Let's place it in (app/modules/apim-v2).

### 2. Update the Stack to use the Module
Now, your apim stack acts as a thin wrapper that calls this module.

Stack: (app/stacks/apim/main.tf)
Stack: app/stacks/apim/variables.tf

The stack variables are what Terraspace actually fills using your .tfvars files.


### 3. The Networking Variables
Add the variables for your networking stack so it can handle different CIDR blocks.

Stack: app/stacks/networking/variables.tf

### 4. Why this structure?
By using this "Module + Stack" approach:

- **Uniformity:** Every APIM instance in your company will follow the same Standard V2 configuration (like the specific VNet delegation).

- **Ease of Updates:** If you need to add a default tag or a security setting to all APIMs, you change it once in the app/modules/apim-v2 directory.

- **Clean Stacks:** Your app/stacks/apim/main.tf stays very short and readable.

## Inbound traffic private for your APIM Standard V2 instance

To make the Inbound traffic private for your APIM Standard V2 instance, we will add an Azure Private Endpoint. This creates a network interface in your VNet, giving APIM a private IP address and allowing you to disable public access entirely.

### 1. Update the Reusable Module
We will update the module to include the Private Endpoint and a Private DNS Zone so that your internal services can resolve the APIM gateway URL to its private IP.

```
# The Private Endpoint itself
resource "azurerm_private_endpoint" "apim_inbound" {
  name                = "${var.apim_name}-pe"
  location            = var.location
  resource_group_name = var.resource_group_name
  subnet_id           = var.inbound_subnet_id

  private_service_connection {
    name                           = "${var.apim_name}-psc"
    private_connection_resource_id = azurerm_api_management.this.id
    subresource_names              = ["Gateway"]
    is_manual_connection           = false
  }
}

# Private DNS Zone for internal resolution
resource "azurerm_private_dns_zone" "apim_dns" {
  name                = "privatelink.azure-api.net"
  resource_group_name = var.resource_group_name
}

# Link DNS to VNet
resource "azurerm_private_dns_zone_virtual_network_link" "dns_link" {
  name                  = "${var.apim_name}-dns-link"
  resource_group_name   = var.resource_group_name
  private_dns_zone_name = azurerm_private_dns_zone.apim_dns.name
  virtual_network_id    = var.vnet_id
}
```

### 2. Updated Variables
We need to pass the inbound_subnet_id and vnet_id into the stack and module.

Module Variables: app/modules/apim-v2/variables.tf
Add these to your module variables:
```
Terraform
variable "inbound_subnet_id" {
  type        = string
  description = "Subnet for the Private Endpoint (Inbound)"
}

variable "vnet_id" {
  type        = string
  description = "The ID of the VNet for DNS linking"
}
```
Stack Variables: app/stacks/apim/variables.tf
Add matching variables to your stack:
```
Terraform
variable "inbound_subnet_id" { type = string }
variable "vnet_id" { type = string }
```
3. The Final Architecture Flow
With this setup, your APIM instance is now fully "network-cloaked":

Inbound: Traffic enters via a Private Endpoint IP. Public access is disabled (public_network_access_enabled = false).

Outbound: APIM talks to your backend databases or internal APIs via VNet Integration on the delegated outbound subnet.

4. Wiring it with Terraspace Layering
In your app/stacks/apim/tfvars/base.tfvars, you can now pull all these IDs from your networking stack dynamically:
```
Ruby
# Pulling IDs from the networking stack output
vnet_id           = "<%= output('networking.vnet_id') %>"
apim_subnet_id    = "<%= output('networking.apim_outbound_subnet_id') %>"
inbound_subnet_id = "<%= output('networking.apim_inbound_subnet_id') %>"
publisher_email   = "admin@example.com"
apim_name         = "apim-<%= Terraspace.env %>"
```

5. Deployment
Run the deployment for all stacks. Terraspace will handle the networking first, then build the APIM with its private links.