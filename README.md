# IaC-Azure-APIM

This project is dedicated to deploying Azure API Management (APIM) with OpenTofu and Terraspace.

## Documentation

- **[APIM Enterprise Best Practices](docs/ENTERPRISE-BEST-PRACTICES.md)** - Comprehensive guide covering architecture, security, operations, and governance for enterprise deployments
- **[APIM Security Best Practices](docs/ECURITY-BEST-PRACTICES.md)** - Security hardening standards including network security, authentication, encryption, and monitoring

## Installation
- **[Install Opentofu](.github/skills/opentofu-install/SKILLS.md)** -
Opentofu is the open source version of Hashicorp's terraform.
- **[Install Terraspace](.github/skills/terraspace-install/SKILLS.md)** -
Wrapper for terraform providing a structured way to keep your infrastructure-as-code (IaC) dry and organized.

## Opentofu Lint
Opentofu has built in validation which terraspace can trigger. 
**Command:** ```terraspace all validate```

### Infrastructure as Code (IaC)
- **Version Control**: Always commit your infrastructure code to git. Track all changes and maintain a clear history.
- **Code Reviews**: Require peer reviews for all infrastructure changes before deployment.
- **DRY Principle**: Use Terraspace's layering and DRY (Don't Repeat Yourself) patterns to avoid code duplication.



## Quick Getting Started Guide

### Provisioning an Azure Resource Group

1. **Clone the Repository**: Start by cloning the repository to your local machine.
2. **Create a feature branch**: Create a new branch for your work to keep changes organized.
   ```Bash
   git checkout -b feature/your-feature-name
3. Verify dependencies are installed.
  - **[opentofu-install](.github/skills/opentofu-install/SKILLS.md)**
  - **[terraspace-install](.github/skills/terraspace-install/SKILLS.md)**  
  - **[azure-cli-install](.github/skills/azure-cli-install/SKILLS.md)**
4. Generate terraspace config folders and files
  - **[terraspace config](.github/skills/tofu-terraspace-setup/SKILLS.md)**
5. Smoke test of terraspace and opentofu
  - **[terraspace-smoketest](.github/skills/smoke-test/SKILLS.md)**
6. Generate a simple project provisioning resource_groups
7. Create a reusable Azure resource group module in: 
  - **[modules folder](app/modules)**
8. Create a main.tf, variables.tf and outputs.tf files in: 
  - **[modules folder](app/modules/resource_group)**
9. Create a coreesponding terraspace stack in:
  - **[stacks folder](app/stacks)**

## Contributing

### Adding a New Feature
1. Create a feature branch: `git checkout -b feature/your-feature`
2. Implement the feature
3. Verify terraspace validate and terraspace plan pass
4. Commit and push
5. Request a Peer Review
6. Merge to main branch



---

**Last Updated**: January 2026
**Review Cycle**: Quarterly
**Next Review**: April 2026