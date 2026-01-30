
# OpenTofu Native Validation

- OpenTofu has a built-in validation command. Terraspace can trigger this for all stacks easily.

Command: 
```
terraspace all validate
```

What it does: This runs tofu validate across all your stacks, checking for internal consistency and syntax errors that a generic linter might miss.