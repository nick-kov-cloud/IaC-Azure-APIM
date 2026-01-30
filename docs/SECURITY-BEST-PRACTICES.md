When working with OpenTofu and Terraspace, your code isn't just a set of instructions; it is a security blueprint. Because you are working with Azure APIM and sensitive network configurations, you should incorporate security at three levels: the State, the Code, and the Secrets.

- **Secrets Management**: Never commit sensitive data (API keys, passwords, connection strings) to the repository. Use Azure Key Vault or environment variables.
- **RBAC**: Implement least-privilege access using Azure Role-Based Access Control.
- **State File Protection**: Keep Terraform/OpenTofu state files in secure remote storage (Azure Storage with encryption enabled).
- **Sensitive Outputs**: Mark sensitive outputs in your configurations and avoid logging them.

## 1. Network Security

### Virtual Network Integration
- Deploy APIM in a dedicated subnet within a Virtual Network
- Use Network Security Groups (NSGs) to restrict inbound/outbound traffic
- Implement Azure Firewall for centralized threat protection
- Use Private Endpoints to access backend services privately

```bash
# Example: Check network configuration
az apim show --resource-group <rg> --name <apim-name> --query virtualNetworkConfiguration
```

### DDoS Protection
- Enable Azure DDoS Protection Standard for production deployments
- Configure Web Application Firewall (WAF) with Azure Application Gateway
- Implement rate limiting and throttling policies
- Monitor DDoS attack metrics in Azure Monitor

## 2. API Authentication & Authorization

### OAuth 2.0 / OpenID Connect
- Enforce OAuth 2.0 for all public APIs (no exceptions)
- Use OpenID Connect for user authentication
- Implement authorization code flow for web applications
- Use client credentials flow for service-to-service communication

### API Key Management
- Rotate API keys every 90 days minimum
- Use subscription-level keys, not master keys, for client access
- Implement key regeneration without downtime
- Track key usage in diagnostic logs

### Mutual TLS (mTLS)
- Enforce mTLS between APIM gateway and backend services
- Use certificate-based authentication for service-to-service calls
- Implement certificate pinning where applicable
- Automate certificate renewal before expiration

## 3. Data Protection

### Encryption in Transit
- Enforce HTTPS/TLS 1.2 minimum (TLS 1.3 recommended)
- Disable HTTP endpoints entirely
- Use strong cipher suites only
- Enable HSTS (HTTP Strict Transport Security) headers

### Encryption at Rest
- Enable encryption for all APIM resources in Azure Storage
- Use Azure-managed keys (default) or customer-managed keys (CMK)
- Encrypt sensitive data in cache (Redis)
- Configure transparent data encryption for backend databases

### Secrets Management
- **Never** store secrets in code, configuration files, or logs
- Use Azure Key Vault exclusively for secret storage
- Implement managed identity for APIM to access Key Vault
- Audit all secret access with Key Vault diagnostic logs

```bash
# Example: Configure managed identity for APIM
az apim update --resource-group <rg> --name <apim-name> \
  --assign-identity [SystemAssigned | UserAssigned]
```

## 4. API Policies & Request Validation

### Input Validation
- Validate request schemas against OpenAPI definitions
- Implement request size limits
- Validate query parameters, headers, and body content
- Reject requests with unexpected content types

### Output Sanitization
- Remove sensitive headers from responses (X-Powered-By, Server version)
- Implement response masking for sensitive data
- Filter sensitive information from error messages
- Avoid exposing internal implementation details

### CORS Configuration
- Explicitly define allowed origins (never use `*` in production)
- Restrict allowed HTTP methods
- Limit allowed headers to necessary ones only
- Set appropriate credentials policy

```xml
<!-- Example: CORS Policy -->
<cors allow-credentials="false">
    <allowed-origins>
        <origin>https://trusted-domain.com</origin>
    </allowed-origins>
    <allowed-methods>
        <method>GET</method>
        <method>POST</method>
    </allowed-methods>
    <allowed-headers>
        <header>Content-Type</header>
        <header>Authorization</header>
    </allowed-headers>
</cors>
```

## 5. Threat Protection

### Rate Limiting & Throttling
- Implement per-subscription rate limits based on tier
- Set operation-level limits for expensive endpoints
- Use sliding window algorithm for accurate rate limiting
- Provide clear rate-limit headers in responses

### IP Restrictions
- Whitelist known client IP addresses where feasible
- Block suspicious or malicious IPs immediately
- Use Azure Firewall for dynamic IP filtering
- Monitor and log all blocked requests

### Web Application Firewall (WAF)
- Deploy Azure Application Gateway with WAF in front of APIM
- Enable OWASP Top 10 protection rules
- Configure custom WAF rules for API-specific threats
- Monitor WAF logs for attack patterns

## 6. Audit & Compliance

### Diagnostic Logging
- Enable diagnostic settings for all APIM instances
- Send logs to Log Analytics workspace
- Configure minimum 90-day retention (1 year for compliance environments)
- Export critical logs to immutable Azure Blob Storage

```bash
# Enable diagnostic logging
az monitor diagnostic-settings create \
  --name apim-diagnostics \
  --resource /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.ApiManagement/service/{name} \
  --logs '[{"category":"GatewayLogs","enabled":true}]' \
  --metrics '[{"category":"AllMetrics","enabled":true}]' \
  --workspace /subscriptions/{sub}/resourcegroups/{rg}/providers/microsoft.operationalinsights/workspaces/{workspace}
```

### Activity Logging
- Audit all APIM configuration changes (policies, APIs, users)
- Track user modifications with timestamps and identities
- Implement immutable audit logs
- Alert on critical changes (policy updates, key regeneration)

### Compliance Monitoring
- Implement Azure Policy for APIM configuration compliance
- Run regular security assessments (Microsoft Defender for Cloud)
- Document compliance requirements (HIPAA, PCI-DSS, GDPR)
- Maintain audit logs for regulatory audits

## 7. Identity & Access Control

### Role-Based Access Control (RBAC)
```bash
# Example: Assign custom role to user
az role assignment create \
  --assignee <user-id> \
  --role "API Management Service Contributor" \
  --resource-group <rg>
```

- Use built-in roles: Contributor, Reader, API Management Service Contributor
- Implement least-privilege principle (minimal required permissions)
- Use Azure AD groups for role management
- Regularly audit and remove unused access

### Multi-Factor Authentication (MFA)
- Enforce MFA for all administrative access
- Require MFA for developer portal access to sensitive APIs
- Use Azure AD Conditional Access policies
- Implement passwordless authentication (Windows Hello, FIDO2)

## 8. Secure Development Practices

### API Design
- Design APIs with security-first mindset
- Implement proper HTTP status codes (401, 403, 429)
- Use meaningful error messages without revealing internals
- Document security requirements in API specifications

### Dependency Management
- Keep OpenTofu, Terraform, and Azure CLI updated
- Scan infrastructure code for security vulnerabilities
- Review policy definitions for security gaps
- Use signed commits for infrastructure changes

### Secrets in IaC
```hcl
# Example: Use Key Vault reference in Terraform
variable "api_key" {
  description = "API key from Key Vault"
  type        = string
  sensitive   = true
}

# Never hardcode secrets - use Key Vault
data "azurerm_key_vault_secret" "api_key" {
  name         = "apim-api-key"
  key_vault_id = azurerm_key_vault.kv.id
}
```

## 9. Monitoring & Incident Response

### Security Alerts
- Monitor failed authentication attempts
- Alert on policy violations or blocked requests
- Track unusual API usage patterns
- Set up alerts for certificate expiration

### Incident Response
- Maintain incident response runbook
- Define escalation procedures
- Implement automated remediation where possible
- Conduct post-incident reviews (RCAs)

## 10. Regular Security Reviews

- **Monthly**: Review access logs and audit trails
- **Quarterly**: Conduct security assessments and penetration testing
- **Semi-annually**: Review and update security policies
- **Annually**: Full security audit and compliance assessment



### 1. Secure Your State File (The "Crown Jewels")
The .tfstate file contains your entire infrastructure map and often plain-text secrets (like connection strings).

State Encryption at Rest: OpenTofu 1.7+ introduced native State Encryption. Use your Azure Key Vault to encrypt the state file before it even hits the storage bucket.
```
Terraform
terraform {
  encryption {
    key_provider "azurekeyvault" "main" {
      vault_uri      = "https://my-vault.azure.net"
      key_name       = "state-encryption-key"
    }
    state {
      method   = method.aes_gcm.main
      enforced = true
    }
  }
}
```

Network-Isolate Storage: Ensure your Azure Storage Account for state has Public Access Disabled and is only accessible via a Private Endpoint from your management VNet or CI/CD runner.

RBAC, not Keys: Never use Storage Account Keys in your backend.tf. Use Azure AD Authentication (use_azuread_auth = true) and assign the "Storage Blob Data Owner" role to your identity.

### 2. Secrets Management (Don't Hardcode)
Even if you use variables, the values can leak into logs or the state file if not handled carefully.

Terraspace Secret Helpers: Use Terraspace’s built-in Ruby helpers to fetch secrets from Azure Key Vault at runtime. This prevents secrets from being committed to your tfvars files.
```
Ruby
# app/stacks/apim/tfvars/base.tfvars
admin_password = "<%= azure_key_vault_secret('apim-admin-password') %>"
```
Mark Variables as Sensitive: In your variables.tf, always use the sensitive = true flag. This prevents OpenTofu from printing the value in your terminal or CI logs.
```
Terraform
variable "admin_password" {
  type      = string
  sensitive = true
}
```
### 3. Infrastructure "Hardening" (Policy as Code)
Secure the resources you are actually deploying, specifically your APIM instance.

- Disable Public Endpoints: For APIM Standard V2, set public_network_access_enabled = false once your Private Endpoint is tested.

- Least Privilege Identity: Enable Managed Identity on the APIM instance so it can talk to other Azure services (like Key Vault) without needing credentials.

- Automated Scanning: Add a Before Hook in Terraspace to run a security scanner like tfsec or checkov.
```
Ruby
# config/hooks/terraspace.rb
before("plan", execute: "tfsec . --severity high")
```
### 4. Architectural Summary
By combining these practices, your traffic and data follow a "Zero Trust" model.

|Feature|	Best Practice|	Benefit|
|---|---|---|
|Authentication|	Managed Identities / OIDC|	No long-lived service principal keys.|
|Networking	|Private Endpoints	|Resources are invisible to the public internet.|
|Validation|	Static Analysis (tfsec)	|Catch misconfigured NSGs or open ports before deployment.|
|Storage|	Object Versioning|	Roll back the state if it gets corrupted or accidentally deleted.|

Implementing an automated security gate ensures that no infrastructure is deployed unless it passes a set of security standards. By using tfsec (a popular static analysis tool for OpenTofu/Terraform), you can catch "high" and "critical" risks like open SSH ports or public storage buckets before they ever reach Azure.

### 🛡️ The Security Hook Configuration
Create or update your config/hooks/terraspace.rb file with the following logic. This hook will run specifically during the plan and up (apply) phases.

```
# config/hooks/terraspace.rb

# 1. Security Scan Hook
# This runs before the plan is generated. 
# If tfsec finds a 'High' or 'Critical' severity issue, it exits with a non-zero code,
# which stops Terraspace from proceeding to the deployment phase.
before("plan", "up",
  execute: "tfsec . --severity high,critical",
  exit_on_fail: true
)

# 2. Secret Leak Detection (Optional but recommended)
# You can also add a hook to check for accidentally hardcoded secrets 
# using tools like 'gitleaks' if you have it installed.
before("all",
  execute: "gitleaks detect --source . --verbose",
  exit_on_fail: true
)

# 3. Success Notification
# Provides a clean confirmation in the console when security checks pass.
after("plan",
  execute: "echo '✅ Security scan passed. Infrastructure plan is compliant.'"
)
```

### 🚦 How it works in practice
When you run terraspace up apim, the following happens:

1. Trigger: Terraspace hits the before("up") hook.
2. Scan: It executes tfsec in your current project directory.
3. Analysis: tfsec checks your APIM and Networking modules against hundreds of Azure best practices (e.g., "Ensure APIM has a managed identity").
4. Enforcement:

  - Pass: If no issues are found, the OpenTofu/Terraform plan begins.
  - Fail: If you accidentally left public_network_access_enabled = true, the scan fails, and Terraspace stops immediately, preventing the deployment.

### 🔑 Handling "False Positives"
Sometimes you have a valid reason to bypass a security rule. You can do this directly in your OpenTofu code using a comment:
```
# tfsec:ignore:azure-api-management-no-public-access
resource "azurerm_api_management" "this" {
  # ... configuration ...
}
```
### 🛠️ Requirements for this Hook
To use this specific hook, you need to have tfsec installed on your machine or CI/CD runner:

- **macOS:** brew install tfsec

- **Linux:** curl -s https://raw.githubusercontent.com/aquasecurity/tfsec/master/scripts/install_linux.sh | bash

- **Windows:** choco install tfsec