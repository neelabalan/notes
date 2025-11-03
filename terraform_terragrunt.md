## Terragrunt

> managing multi-environment infrastructure

## The problem with Terraform alone

When managing infrastructure across multiple environments (production, staging, development), teams often face several challenges even when using Terraform's built-in features like workspaces and modules.

### Challenges with Terraform Workspaces

Terraform workspaces allow you to manage multiple state files within a single configuration, but they have limitations:

- All workspaces share the same backend configuration, making it difficult to isolate environments completely
- Variables cannot be easily customized per workspace without complex conditional logic
- No enforcement of environment-specific configurations
- State files for all workspaces are stored in the same backend location, increasing blast radius
- Difficult to implement different resource counts or configurations per environment
- Team members can accidentally switch workspaces and apply changes to the wrong environment

### Challenges with Terraform Modules

While modules promote code reusability, they introduce their own complexities:

- Repetitive backend configurations must be defined in each environment's root module
- Provider configurations need to be duplicated across environments
- No built-in way to keep configurations DRY (Don't Repeat Yourself)
- Managing variable files for different environments becomes cumbersome
- Dependency management between modules is manual and error-prone
- Updating module versions across multiple environments requires touching many files

## How Terragrunt solves these problems

Terragrunt acts as a thin wrapper around Terraform that provides additional tooling for managing configurations across multiple environments.

### Key Benefits

1. DRY configuration: Terragrunt allows you to define common configurations once and inherit them across environments using a hierarchical structure. Backend configurations, provider settings, and common variables can be defined at the root level.

2. Environment isolation: Each environment gets its own directory with its own terragrunt.hcl file. State files are automatically organized and isolated per environment, reducing the risk of cross-environment contamination.

3. Dependency management: Terragrunt can define and enforce dependencies between modules. If module B depends on module A, Terragrunt ensures A is applied before B, and handles passing outputs between them automatically.

4. Automatic remote state management: Terragrunt can automatically configure and create remote state backends, including S3 buckets and DynamoDB tables for state locking, without manual setup.

5. Built-in best practices: Features like automatic retry on transient errors, execution hooks for running scripts before/after Terraform commands, and validation of configurations help enforce operational best practices.

6. Bulk operations: Run terraform commands across multiple modules simultaneously with a single command, useful for updates affecting multiple services.

## Practical Example Structure

A typical Terragrunt setup might look like:

```
infrastructure/
├── terragrunt.hcl (root config)
├── prod/
│   ├── vpc/
│   │   └── terragrunt.hcl
│   ├── database/
│   │   └── terragrunt.hcl
│   └── app/
│       └── terragrunt.hcl
├── staging/
│   ├── vpc/
│   │   └── terragrunt.hcl
│   └── app/
│       └── terragrunt.hcl
└── dev/
    └── app/
        └── terragrunt.hcl
```

Each environment references the same Terraform modules but with environment-specific variables. The root terragrunt.hcl defines common backend and provider configurations that all child configurations inherit.

## Comparison of Approaches

| Aspect | Pure Terraform | Terraform Workspaces | Terraform Modules | Terragrunt |
|--------|---------------|---------------------|-------------------|------------|
| Code Reusability | Limited, lots of duplication | Shared code, separate state | High with modules | Highest, DRY principles enforced |
| Environment Isolation | Manual separation of directories | Shared config, separate state | Separate root modules per env | Strong, separate directories and state |
| Backend Configuration | Repeated in each config | Shared for all workspaces | Repeated per environment | Defined once, inherited |
| State Management | Manual organization | Automatic per workspace | Manual per root module | Automatic with best practices |
| Variable Management | Separate .tfvars per env | Conditional logic or separate files | Separate .tfvars per root | Hierarchical, inherited variables |
| Dependency Handling | Manual with data sources | Manual with data sources | Manual with data sources | Automatic with dependency blocks |
| Risk of Cross-Environment Changes | Medium to high | High, easy to switch workspace | Medium, separate roots | Low, strong isolation |
| Learning Curve | Baseline | Minimal addition | Moderate | Steeper, additional tool |
| Operational Overhead | High, manual processes | Medium | Medium to high | Lower with automation |
| Module Version Management | Manual in each location | Manual in each location | Manual per root module | Centralized, easier updates |
| Bulk Operations | Manual scripts required | Limited support | Manual scripts required | Built-in support |
| Remote State Setup | Manual configuration | Manual configuration | Manual configuration | Automatic generation |
| Provider Configuration | Repeated everywhere | Shared but inflexible | Repeated per root | Defined once, inherited |