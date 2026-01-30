# Azure API Management (APIM) V2 Developer Tier

This guide provides a production-ready blueprint for provisioning the Azure API Management (APIM) V2 Developer Tier using OpenTofu and Terraspace.

The Developer V2 tier is ideal for testing because it mirrors the networking capabilities of the Standard V2 tier (VNet Integration/Private Endpoints) at a lower price point.

Reference:
- **[APIM Design Guide](../.github/instructions/apim-design.md)**

### 🏗️ 1. Architecture Strategy: "Network-Cloaked" APIM
We will implement the Hub-and-Spoke security model. Your APIM will have:

Inbound: Private Endpoint (No public internet access).

Outbound: VNet Integration (Access to private backends/databases).

Identity: System-assigned Managed Identity.

### 2. The Reusable Module (app/modules/apim)
This module follows security best practices by disabling public access and enabling Managed Identity.
```
main.tf
Terraform
resource "azurerm_api_management" "apim" {
  name                = var.name
  location            = var.location
  resource_group_name = var.resource_group_name
  publisher_name      = var.publisher_name
  publisher_email     = var.publisher_email
  sku_name            = "DeveloperV2_1" # V2 Tier

  # Security: Disable public access
  public_network_access_enabled = false

  identity {
    type = "SystemAssigned"
  }
}

# Outbound VNet Integration
resource "azurerm_api_management_network_configuration" "outbound" {
  api_management_id = azurerm_api_management.apim.id
  subnet_id          = var.outbound_subnet_id
}

# Inbound Private Endpoint
resource "azurerm_private_endpoint" "inbound" {
  name                = "${var.name}-pe"
  location            = var.location
  resource_group_name = var.resource_group_name
  subnet_id           = var.inbound_subnet_id

  private_service_connection {
    name                           = "${var.name}-link"
    private_connection_resource_id = azurerm_api_management.apim.id
    subresource_names              = ["Gateway"]
    is_manual_connection           = false
  }
}
```
### 3. Security Best Practices Implementation
#### A. Secret Management
Use Azure Key Vault for the publisher email or any sensitive certificates. Never hardcode them in tfvars.
```
Ruby
# app/stacks/apim/tfvars/base.tfvars
publisher_email = "<%= azure_key_vault_secret('admin-email') %>"
```

#### B. OpenTofu Variables (variables.tf)
Always mark sensitive data and provide descriptions for auditability.
```
Terraform
variable "publisher_email" {
  type      = string
  sensitive = true
}

variable "inbound_subnet_id" {
  type        = string
  description = "Must be a dedicated subnet or have enough IP space for Private Endpoints."
}
```
### 4. Automated Governance (Terraspace Hooks)
To enforce security, use the tfsec hook we configured earlier. It will catch common mistakes like forgetting to enable Managed Identity.
```
Ruby
# config/hooks/terraspace.rb
before("up", "plan", execute: "tfsec . --severity high")
```
### 5. Deployment Guide
#### Step 1: Initialize Networking
Deploy your VNet and delegated subnets first.
```
Bash
terraspace up networking
```
#### Step 2: Deploy APIM
Terraspace will automatically pull the subnet_ids from your networking stack.
```
Bash
terraspace up apim
```
### Step 3: Post-Deployment Verification
Check that the Managed Identity has been created:
```
Bash
az apim show --name my-apim --resource-group my-rg --query "identity"
```
## 🛡️ Summary Checklist for Security
[  ] Identity: Managed Identity is enabled (No Service Principal keys needed).

[  ] Network: public_network_access_enabled set to false.

[  ] Inbound: Traffic routed through Private Endpoint.

[  ] Outbound: VNet integration used for backend communication.

[  ] State: Remote state is encrypted in an Azure Storage Account.


### APPLIES TO: Standard v2 | Premium v2


# Integrate an Azure API Management instance with a private virtual network for outbound connections

This article guides you through the process of configuring virtual network integration for your Standard v2 or Premium v2 (preview) Azure API Management instance. With virtual network integration, your instance can make outbound requests to APIs that are isolated in a single connected virtual network or any peered virtual network, as long as network connectivity is properly configured.

When an API Management instance is integrated with a virtual network for outbound requests, the gateway and developer portal endpoints remain publicly accessible. The API Management instance can reach both public and network-isolated backend services.



**Important**

- Outbound virtual network integration described in this article is available only for API Management instances in the Standard v2 and Premium v2 tiers. For networking options in the different tiers, see Use a virtual network with Azure API Management.
- You can enable virtual network integration when you create an API Management instance in the Standard v2 or Premium v2 tier, or after the instance is created.
- Currently, you can't switch between virtual network injection and virtual network integration for a Premium v2 instance.

## Prerequisites

- An Azure API Management instance in the Standard v2 or Premium v2 pricing tier
- (Optional) For testing, a sample backend API hosted within a different subnet in the virtual network. For example, see Tutorial: Establish Azure Functions private site access.
- A virtual network with a subnet where your API Management backend APIs are hosted. See the following sections for requirements and recommendations for the virtual network and subnet.

### Network location
The virtual network must be in the same region and Azure subscription as the API Management instance.

### Dedicated subnet
The subnet used for virtual network integration can only be used by a single API Management instance. It can't be shared with another Azure resource.

### Subnet size
- Minimum: /27 (32 addresses)
- Recommended: /24 (256 addresses) - to accommodate scaling of API Management instance

### Network security group
A network security group (NSG) must be associated with the subnet. To set up a network security group, see Create a network security group.

- Configure the following rule to allow outbound access to Azure Storage, which is a dependency for API Management.
- Configure other outbound rules you need for the gateway to reach your API backends.
- Configure other NSG rules to meet your organization’s network access requirements. For example, NSG rules can also be used to block outbound traffic to the internet and allow access only to resources in your virtual network.


Direction	Source	      Source port ranges	Destination	Destination port ranges	Protocol	Action	Purpose
Outbound	VirtualNetwork	*	                Storage	     443	                  TCP	    Allow	Dependency on Azure Storage


**Important**

- Inbound NSG rules do not apply when a v2 tier instance is integrated in a virtual network for private outbound access. To enforce inbound NSG rules, use virtual network injection instead of integration.
- This differs from networking in the classic Premium tier, where inbound NSG rules are enforced in both external and internal virtual network injection modes. Learn more


## Subnet delegation
The subnet needs to be delegated to the Microsoft.Web/serverFarms service.

**Note**

You might need to register the Microsoft.Web/serverFarms resource provider in the subscription so that you can delegate the subnet to the service, even if you see it on the list of available services in the subnet delegation setup in the portal.

### Permissions
You must have at least the following role-based access control permissions on the subnet or at a higher level to configure virtual network integration:

**Action**	                                             **Description**
Microsoft.Network/virtualNetworks/read	                Read the virtual network definition
Microsoft.Network/virtualNetworks/subnets/read	        Read a virtual network subnet definition
Microsoft.Network/virtualNetworks/subnets/join/action	Joins a virtual network



## Configure virtual network integration
This section guides you through the process to configure external virtual network integration for an existing Azure API Management instance. You can also configure virtual network integration when you create a new API Management instance.

In the Azure portal, navigate to your API Management instance.
In the left menu, under Deployment + Infrastructure, select Network > Edit.
On the Network configuration page, under Outbound features, select Enable virtual network integration.
Select the virtual network and the delegated subnet that you want to integrate.
Select Save. The virtual network is integrated.
