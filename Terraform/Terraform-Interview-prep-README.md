# Terraform Interview Crash Course — Detailed Fresher-Friendly Edition

> Practical Terraform interview preparation with beginner explanations, commands, HCL examples, Azure examples, state management, modules, CI/CD, troubleshooting, and scenario-based questions.

---

# Index

1. [What is Terraform?](#1-what-is-terraform)
2. [Infrastructure as Code](#2-infrastructure-as-code)
3. [Terraform Workflow](#3-terraform-workflow)
4. [Terraform Files](#4-terraform-files)
5. [HCL Basics](#5-hcl-basics)
6. [Providers](#6-providers)
7. [Resources](#7-resources)
8. [Terraform init](#8-terraform-init)
9. [Terraform fmt](#9-terraform-fmt)
10. [Terraform validate](#10-terraform-validate)
11. [Terraform plan](#11-terraform-plan)
12. [Terraform apply](#12-terraform-apply)
13. [Terraform destroy](#13-terraform-destroy)
14. [Variables](#14-variables)
15. [tfvars](#15-tfvars)
16. [Outputs](#16-outputs)
17. [Locals](#17-locals)
18. [Data Sources](#18-data-sources)
19. [Terraform State](#19-terraform-state)
20. [Remote State with Azure Storage](#20-remote-state-with-azure-storage)
21. [State Locking](#21-state-locking)
22. [Terraform State Commands](#22-terraform-state-commands)
23. [Import Existing Resources](#23-import-existing-resources)
24. [Dependencies and depends_on](#24-dependencies-and-depends_on)
25. [count](#25-count)
26. [for_each](#26-for_each)
27. [count vs for_each](#27-count-vs-for_each)
28. [Conditional Expressions](#28-conditional-expressions)
29. [Functions and Expressions](#29-functions-and-expressions)
30. [Lifecycle Rules](#30-lifecycle-rules)
31. [Resource Replacement](#31-resource-replacement)
32. [Modules](#32-modules)
33. [Workspaces](#33-workspaces)
34. [Sensitive Values and Secrets](#34-sensitive-values-and-secrets)
35. [Terraform Lock File](#35-terraform-lock-file)
36. [Azure Authentication](#36-azure-authentication)
37. [Terraform with Azure DevOps CI/CD](#37-terraform-with-azure-devops-cicd)
38. [Drift](#38-drift)
39. [Troubleshooting](#39-troubleshooting)
40. [Production Best Practices](#40-production-best-practices)
41. [Scenario-Based Interview Questions](#41-scenario-based-interview-questions)
42. [Rapid-Fire Interview Q&A](#42-rapid-fire-interview-qa)
43. [Last-Minute Cheat Sheet](#43-last-minute-cheat-sheet)

---

# 1. What is Terraform?

Terraform is an **Infrastructure as Code (IaC)** tool.

Instead of manually creating infrastructure through a cloud portal, you describe the desired infrastructure in configuration files.

Example:

```hcl
resource "azurerm_resource_group" "demo" {
  name     = "rg-terraform-demo"
  location = "Central India"
}
```

Terraform can then create and manage that resource.

### Why is Terraform useful?

Terraform provides:

- Repeatable infrastructure deployments
- Version-controlled infrastructure
- Automation
- Consistency between environments
- Dependency management
- Change previews using `terraform plan`
- Support for many providers such as Azure, AWS, GCP, Kubernetes and GitHub

### Interview question

**Is Terraform imperative or declarative?**

Terraform is primarily **declarative**.

You describe the desired end state:

```hcl
resource "azurerm_resource_group" "demo" {
  name     = "rg-demo"
  location = "Central India"
}
```

Terraform determines the actions required to reach that state.

---

# 2. Infrastructure as Code

Without IaC:

```text
Engineer
   |
Azure Portal
   |
Click -> Create VNet
Click -> Create Storage
Click -> Configure App
```

With Terraform:

```text
Git Repository
     |
Terraform Configuration
     |
terraform plan
     |
terraform apply
     |
Azure
```

Benefits include reviewable pull requests, reusable code and consistent environments.

---

# 3. Terraform Workflow

The core workflow is:

```text
Write configuration
       |
       v
terraform init
       |
       v
terraform fmt
       |
       v
terraform validate
       |
       v
terraform plan
       |
       v
Review the plan
       |
       v
terraform apply
```

Typical commands:

```bash
terraform init
terraform fmt -check
terraform validate
terraform plan
terraform apply
```

Destroy only when intentionally required:

```bash
terraform destroy
```

---

# 4. Terraform Files

Common files:

```text
terraform-project/
├── main.tf
├── variables.tf
├── outputs.tf
├── providers.tf
├── backend.tf
├── terraform.tfvars
└── .terraform.lock.hcl
```

These names are conventions, not requirements. Terraform loads `.tf` files in the working directory as one configuration.

Typical responsibilities:

```text
main.tf              resources/modules
variables.tf         input variable declarations
outputs.tf           output declarations
providers.tf         provider requirements/configuration
backend.tf           backend configuration
terraform.tfvars     variable values
.terraform.lock.hcl  selected provider versions/checksums
```

---

# 5. HCL Basics

Terraform configurations use HCL.

Example block:

```hcl
resource "azurerm_resource_group" "example" {
  name     = "rg-demo"
  location = "Central India"
}
```

General structure:

```text
resource "RESOURCE_TYPE" "LOCAL_NAME" {
  argument = value
}
```

Reference it with:

```hcl
azurerm_resource_group.example.name
```

Comments:

```hcl
# Comment

// Also a comment

/*
Multi-line
comment
*/
```

---

# 6. Providers

A provider plugin allows Terraform to communicate with an API/platform.

Example Azure provider requirement:

```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 4.0"
    }
  }
}

provider "azurerm" {
  features {}
}
```

### Why needed?

Terraform itself does not contain all Azure resource implementations. The AzureRM provider supplies Azure-specific resource and data-source types.

### Interview question

**What does `terraform init` do with providers?**

It resolves and installs the required provider plugins according to configuration and the dependency lock file.

---

# 7. Resources

A resource tells Terraform to manage an infrastructure object.

```hcl
resource "azurerm_resource_group" "example" {
  name     = "rg-interview-demo"
  location = "Central India"
}
```

Resource address:

```text
azurerm_resource_group.example
```

Example storage account:

```hcl
resource "azurerm_storage_account" "example" {
  name                     = "stinterviewdemo1234"
  resource_group_name      = azurerm_resource_group.example.name
  location                 = azurerm_resource_group.example.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
}
```

Notice that the storage account references the resource group. Terraform can infer a dependency from this reference.

---

# 8. Terraform init

```bash
terraform init
```

### What does it do?

Initializes a Terraform working directory.

It commonly:

- Initializes the backend
- Resolves/downloads provider plugins
- Downloads referenced modules
- Prepares the directory for Terraform operations

### When is it needed?

Commonly after:

- Cloning a Terraform repository
- Adding/changing provider requirements
- Adding/changing modules
- Changing backend configuration

If backend configuration changes, options such as these may be relevant:

```bash
terraform init -reconfigure
```

or during an intentional backend migration:

```bash
terraform init -migrate-state
```

Read the proposed action carefully before migrating production state.

---

# 9. Terraform fmt

```bash
terraform fmt
```

Formats Terraform files into canonical style.

Check formatting without modifying:

```bash
terraform fmt -check
```

Recursive:

```bash
terraform fmt -recursive
```

### Why needed?

Consistent formatting makes code reviews easier and is useful as a CI validation step.

---

# 10. Terraform validate

```bash
terraform validate
```

Checks whether the configuration is syntactically valid and internally consistent.

### Important interview distinction

`validate` does **not** mean:

> "Azure definitely accepts and can deploy everything."

It primarily validates the Terraform configuration. `plan` performs broader evaluation in context and may contact providers/APIs as needed.

---

# 11. Terraform plan

```bash
terraform plan
```

### What does it do?

Creates an execution plan showing proposed changes.

Common symbols:

```text
+ create
- destroy
~ update in-place
-/+ destroy and recreate
```

Example:

```text
# azurerm_resource_group.example will be created
+ resource "azurerm_resource_group" "example" {
    + name = "rg-demo"
  }
```

Save a plan:

```bash
terraform plan -out=tfplan
```

Inspect it:

```bash
terraform show tfplan
```

Apply that exact saved plan:

```bash
terraform apply tfplan
```

### Interview best practice

Always review a production plan carefully, especially any destruction or replacement.

---

# 12. Terraform apply

```bash
terraform apply
```

Terraform generates a plan and asks for approval before executing it.

Apply a saved plan:

```bash
terraform apply tfplan
```

Auto approval exists:

```bash
terraform apply -auto-approve
```

Use automation controls carefully. Production changes should have appropriate review/approval gates.

---

# 13. Terraform destroy

```bash
terraform destroy
```

Terraform proposes destruction of resources managed by the configuration/state.

Preview:

```bash
terraform plan -destroy
```

### Interview point

Never treat `destroy` casually in shared or production environments.

---

# 14. Variables

Variables make configurations reusable.

`variables.tf`:

```hcl
variable "environment" {
  description = "Deployment environment"
  type        = string
  default     = "dev"
}
```

Use:

```hcl
resource "azurerm_resource_group" "example" {
  name     = "rg-${var.environment}"
  location = "Central India"
}
```

Another example:

```hcl
variable "location" {
  type        = string
  description = "Azure region"
}
```

Types include:

```text
string
number
bool
list(...)
set(...)
map(...)
object(...)
tuple(...)
```

---

# 15. tfvars

Variable declarations:

```hcl
variable "environment" {
  type = string
}
```

Values in `terraform.tfvars`:

```hcl
environment = "dev"
```

Custom file:

```text
dev.tfvars
prod.tfvars
```

Use:

```bash
terraform plan -var-file="dev.tfvars"
```

Command-line variable:

```bash
terraform plan -var="environment=dev"
```

Environment variable form:

```bash
export TF_VAR_environment=dev
```

### Security warning

Do not commit secrets in `.tfvars` just because the file format supports variables.

---

# 16. Outputs

Outputs expose useful values after evaluation/apply.

```hcl
output "resource_group_name" {
  value = azurerm_resource_group.example.name
}
```

View outputs:

```bash
terraform output
```

Specific output:

```bash
terraform output resource_group_name
```

Machine-readable:

```bash
terraform output -json
```

---

# 17. Locals

Locals reduce repetition inside a configuration/module.

```hcl
locals {
  common_tags = {
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
}
```

Use:

```hcl
tags = local.common_tags
```

### Variable vs local

```text
variable -> input from outside the module
local    -> internally calculated/reused value
```

---

# 18. Data Sources

Resources manage infrastructure.

Data sources read information.

Example:

```hcl
data "azurerm_resource_group" "existing" {
  name = "rg-existing"
}
```

Reference:

```hcl
data.azurerm_resource_group.existing.location
```

### Interview question

**Resource vs data source?**

```text
resource -> Terraform manages lifecycle of an object
data     -> Terraform reads information from an existing object/API
```

---

# 19. Terraform State

Terraform state is one of the most important interview topics.

Terraform maintains a mapping between configuration addresses and real infrastructure, plus attributes needed for planning.

Default local state file:

```text
terraform.tfstate
```

Concept:

```text
Terraform Configuration
         |
         v
Terraform State
         |
         v
Real Infrastructure
```

### Why state?

Terraform needs to understand which real object corresponds to:

```text
azurerm_resource_group.example
```

### Important security point

State may contain sensitive values. Treat state as sensitive data even if variables/outputs are marked `sensitive`.

### Why not use local state for a team?

Problems include:

- Hard to share safely
- Concurrent execution risk
- Local-machine loss
- Poor central governance
- Potential exposure

Use an appropriate remote backend for team environments.

---

# 20. Remote State with Azure Storage

A common Azure backend is Azure Blob Storage.

Example:

```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "sttfstateexample"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
  }
}
```

Then:

```bash
terraform init
```

### Why remote state?

- Shared team state
- Centralized access controls
- Durability
- Backend-supported locking
- Better CI/CD integration

### Important security practices

Protect the storage account using appropriate RBAC/networking, avoid embedding credentials in source code, and enable recovery/versioning controls appropriate to your organization.

---

# 21. State Locking

State locking prevents conflicting state-writing operations from running at the same time when the backend supports locking.

Example problem:

```text
Engineer A -> terraform apply
                     \
                      Same state
                     /
Engineer B -> terraform apply
```

Without coordination, concurrent writes could corrupt or produce inconsistent state.

The AzureRM backend supports state locking using Azure Blob mechanisms.

### Interview question

**Why is state locking important?**

To prevent multiple writers from changing the same state simultaneously.

---

# 22. Terraform State Commands

List resources:

```bash
terraform state list
```

Inspect a resource:

```bash
terraform state show azurerm_resource_group.example
```

Move an address:

```bash
terraform state mv OLD_ADDRESS NEW_ADDRESS
```

Remove from Terraform state without deleting the real object:

```bash
terraform state rm ADDRESS
```

### Warning

State commands can be dangerous. Back up/protect state and understand the effect before modifying production state.

---

# 23. Import Existing Resources

If an Azure resource already exists and you want Terraform to manage it, import associates it with a Terraform resource address.

Configuration:

```hcl
resource "azurerm_resource_group" "existing" {
  name     = "rg-existing"
  location = "Central India"
}
```

Traditional CLI import:

```bash
terraform import azurerm_resource_group.existing "/subscriptions/<subscription-id>/resourceGroups/rg-existing"
```

Modern Terraform also supports configuration-driven `import` blocks:

```hcl
import {
  to = azurerm_resource_group.existing
  id = "/subscriptions/<subscription-id>/resourceGroups/rg-existing"
}
```

Then review:

```bash
terraform plan
```

### Important interview point

Import brings the object under state management; you still need configuration that represents the desired resource.

---

# 24. Dependencies and depends_on

Terraform usually detects dependencies through references.

```hcl
resource "azurerm_storage_account" "example" {
  resource_group_name = azurerm_resource_group.example.name
}
```

This creates an **implicit dependency**.

Explicit dependency:

```hcl
depends_on = [
  azurerm_resource_group.example
]
```

### Interview answer

Prefer natural references/implicit dependencies. Use `depends_on` when there is a real dependency Terraform cannot infer from expressions.

---

# 25. count

Create multiple similar instances:

```hcl
resource "azurerm_resource_group" "example" {
  count = 3

  name     = "rg-demo-${count.index}"
  location = "Central India"
}
```

Addresses look like:

```text
azurerm_resource_group.example[0]
azurerm_resource_group.example[1]
```

Conditional creation:

```hcl
count = var.create_resource ? 1 : 0
```

---

# 26. for_each

Useful when resources have stable unique keys.

```hcl
variable "resource_groups" {
  type = set(string)

  default = [
    "rg-dev",
    "rg-test",
    "rg-prod"
  ]
}

resource "azurerm_resource_group" "example" {
  for_each = var.resource_groups

  name     = each.value
  location = "Central India"
}
```

Addresses:

```text
azurerm_resource_group.example["rg-dev"]
azurerm_resource_group.example["rg-prod"]
```

Map example:

```hcl
variable "resource_groups" {
  type = map(string)

  default = {
    dev  = "Central India"
    prod = "South India"
  }
}

resource "azurerm_resource_group" "example" {
  for_each = var.resource_groups

  name     = "rg-${each.key}"
  location = each.value
}
```

---

# 27. count vs for_each

Common interview question.

```text
count
-----
Index-based instances
Useful for simple repeated/conditional resources

for_each
--------
Key-based instances
Useful when instances have meaningful stable identities
```

Why stable keys matter:

With list/index-based designs, removing/reordering an element can change instance addresses. `for_each` with stable keys often makes lifecycle behavior clearer.

---

# 28. Conditional Expressions

Syntax:

```hcl
condition ? true_value : false_value
```

Example:

```hcl
account_tier = var.environment == "prod" ? "Premium" : "Standard"
```

Conditional resource:

```hcl
count = var.environment == "prod" ? 1 : 0
```

---

# 29. Functions and Expressions

Examples:

```hcl
upper(var.environment)
lower(var.environment)
length(var.resource_groups)
lookup(var.tags, "Owner", "unknown")
```

String interpolation:

```hcl
name = "rg-${var.environment}"
```

Modern HCL can also directly assign expressions without interpolation when the entire value is an expression.

---

# 30. Lifecycle Rules

Terraform's `lifecycle` block can customize resource lifecycle behavior.

## prevent_destroy

```hcl
lifecycle {
  prevent_destroy = true
}
```

Helps protect a resource from destruction through Terraform while the rule is present.

## create_before_destroy

```hcl
lifecycle {
  create_before_destroy = true
}
```

Requests creation of replacement before destruction when the resource/provider can support that ordering.

## ignore_changes

```hcl
lifecycle {
  ignore_changes = [
    tags
  ]
}
```

Use carefully. Ignoring changes means Terraform intentionally stops reconciling those selected attributes, which can hide meaningful drift.

---

# 31. Resource Replacement

Older workflows commonly referenced:

```bash
terraform taint RESOURCE
```

Current Terraform workflows generally prefer planning an explicit replacement:

```bash
terraform apply -replace="azurerm_resource_group.example"
```

Preview first:

```bash
terraform plan -replace="ADDRESS"
```

### Why replacement?

Useful when an object must be recreated even though Terraform does not otherwise detect a configuration change requiring replacement.

---

# 32. Modules

Modules are reusable Terraform configurations.

Example structure:

```text
terraform/
├── main.tf
├── variables.tf
└── modules/
    └── resource-group/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

Module:

```hcl
variable "name" {
  type = string
}

variable "location" {
  type = string
}

resource "azurerm_resource_group" "this" {
  name     = var.name
  location = var.location
}

output "name" {
  value = azurerm_resource_group.this.name
}
```

Call:

```hcl
module "resource_group" {
  source = "./modules/resource-group"

  name     = "rg-dev"
  location = "Central India"
}
```

### Why modules?

- Reuse
- Standardization
- Reduced duplication
- Organizational conventions
- Easier maintenance

### Interview question

**What is the root module?**

The configuration in the directory where you run Terraform is the root module. Modules called from it are child modules.

---

# 33. Workspaces

List:

```bash
terraform workspace list
```

Create:

```bash
terraform workspace new dev
```

Select:

```bash
terraform workspace select dev
```

Current:

```bash
terraform workspace show
```

Use name:

```hcl
name = "rg-${terraform.workspace}"
```

### Interview caution

CLI workspaces provide separate state instances for the same configuration, but they are not automatically a complete environment-isolation strategy. Many organizations use separate state keys/backends, directories, repositories or deployment boundaries for stronger production isolation.

---

# 34. Sensitive Values and Secrets

Variable:

```hcl
variable "password" {
  type      = string
  sensitive = true
}
```

Output:

```hcl
output "password" {
  value     = var.password
  sensitive = true
}
```

### Critical interview point

`sensitive = true` primarily redacts values from normal CLI/UI output. It does **not** guarantee the value is absent from Terraform state.

Avoid:

```hcl
password = "SuperSecret123"
```

in source code.

Prefer secure CI/CD secret stores, workload identity/service connections, Azure Key Vault integrations where appropriate, and minimal secret propagation.

---

# 35. Terraform Lock File

File:

```text
.terraform.lock.hcl
```

Terraform records selected provider versions and checksums here.

Generally commit it to version control for root configurations so team members/CI use consistent provider selections.

Do not confuse:

```text
.terraform.lock.hcl -> dependency/provider lock file
state locking       -> prevents concurrent state writes
```

They are different concepts.

---

# 36. Azure Authentication

Terraform AzureRM deployments require Azure authentication.

Common automation approach: an Azure identity such as a service principal or workload identity with least-privilege RBAC.

Environment-variable style service-principal authentication can use values such as:

```bash
export ARM_CLIENT_ID="..."
export ARM_CLIENT_SECRET="..."
export ARM_TENANT_ID="..."
export ARM_SUBSCRIPTION_ID="..."
```

Do not hardcode secrets into `.tf` files.

In CI/CD, prefer the authentication mechanism supported by your platform and organization, ideally avoiding long-lived client secrets when workload/federated identity is available.

---

# 37. Terraform with Azure DevOps CI/CD

Typical flow:

```text
Pull Request
    |
terraform fmt -check
    |
terraform validate
    |
terraform plan
    |
Review / Approval
    |
Merge / Deployment stage
    |
terraform apply saved-plan
```

Illustrative Azure Pipelines structure:

```yaml
steps:
- checkout: self

- script: terraform init
  displayName: Terraform Init

- script: terraform fmt -check
  displayName: Terraform Format Check

- script: terraform validate
  displayName: Terraform Validate

- script: terraform plan -out=tfplan
  displayName: Terraform Plan
```

In a real pipeline, authentication/backend access must be securely configured. Production apply should normally be controlled by branch policies, environments, approvals and appropriate identities.

### Interview point

Avoid letting every pull request automatically apply infrastructure to production.

---

# 38. Drift

Drift means real infrastructure differs from the configuration/state expectations, often because of out-of-band/manual changes.

Example:

```text
Terraform says:
SKU = Standard

Someone changes Azure manually:
SKU = Premium
```

A subsequent refresh/plan may detect the difference and propose actions based on the configuration.

### Interview question

**How do you detect drift?**

Run:

```bash
terraform plan
```

For automation specifically checking whether remote objects differ from state without proposing configuration changes, Terraform also provides:

```bash
terraform plan -refresh-only
```

Always review the plan before deciding whether Terraform should revert the manual change or the code should be updated.

---

# 39. Troubleshooting

## `terraform init` fails

Investigate:

- Internet/provider registry connectivity
- Authentication for private module sources
- Backend credentials/access
- Provider constraints
- Proxy/firewall
- Backend configuration

## `terraform validate` fails

Read the exact error and check:

- HCL syntax
- Invalid references
- Required arguments
- Type mismatches

## `terraform plan` fails with authorization

Check:

- Which identity Terraform is using
- Subscription/tenant
- RBAC role
- Resource scope
- Backend access separately from deployment access

## State lock error

Do not immediately force-unlock.

First determine whether another Terraform operation is genuinely running.

Only if you are certain the lock is stale and understand the backend state:

```bash
terraform force-unlock LOCK_ID
```

## Resource exists already

Options depend on intent:

- Import it into Terraform
- Use a data source if Terraform should only read it
- Rename/create a different object
- Remove conflicting unmanaged infrastructure only if intentionally approved

## Terraform wants to destroy something unexpectedly

**Stop. Do not apply.**

Investigate:

```bash
terraform plan
terraform state list
terraform state show ADDRESS
```

Check:

- Resource address changed
- `count`/`for_each` keys changed
- Resource renamed/moved
- State/backend mismatch
- Wrong workspace
- Wrong `.tfvars`
- Provider changes
- ForceNew/replacement-causing argument changes

Modern Terraform supports `moved` blocks for many refactors:

```hcl
moved {
  from = azurerm_resource_group.old
  to   = azurerm_resource_group.new
}
```

---

# 40. Production Best Practices

1. Use remote state.
2. Protect backend access.
3. Use state locking.
4. Treat state as sensitive.
5. Pin/constraint provider versions appropriately.
6. Commit `.terraform.lock.hcl` for root configurations.
7. Use modules for repeatable patterns.
8. Run `fmt`, `validate` and `plan` in CI.
9. Review plans before apply.
10. Use least-privilege identities.
11. Do not hardcode secrets.
12. Separate environments appropriately.
13. Protect production applies with approvals/policies.
14. Avoid unnecessary manual cloud changes.
15. Back up/version remote state appropriately.
16. Review all replacements/destructions carefully.

---

# 41. Scenario-Based Interview Questions

## Scenario 1: Two engineers run apply simultaneously

**Answer:**

Use a remote backend that supports state locking. One operation should acquire the lock; the other should wait/fail rather than concurrently write the same state.

---

## Scenario 2: Someone manually changed an Azure resource

**Answer:**

This is configuration drift. Run:

```bash
terraform plan
```

Review whether Terraform proposes restoring the declared configuration. Decide whether the manual change was valid; if so, update Terraform code appropriately rather than blindly reverting it.

---

## Scenario 3: State file is lost

This can be serious because Terraform loses its mapping/knowledge of managed objects.

For remote state, restore from the backend's recovery/versioning mechanism if available. Otherwise, recovery may involve rebuilding state/importing existing infrastructure carefully.

Do **not** simply run apply and hope Terraform recognizes everything automatically.

---

## Scenario 4: Existing Azure resource must be managed

Define the corresponding resource and import it:

```bash
terraform import ADDRESS AZURE_RESOURCE_ID
```

Then:

```bash
terraform plan
```

Reconcile configuration until the plan reflects the intended desired state.

---

## Scenario 5: Plan shows unexpected destruction

Do not apply.

Verify:

```bash
terraform workspace show
terraform state list
terraform plan
```

Then inspect variable files, backend/state, resource addresses, keys and recent code/provider changes.

---

## Scenario 6: Need dev/test/prod

Possible strategies include:

- Separate root configurations/directories
- Separate state keys/backends
- Reusable modules
- Environment-specific `.tfvars`
- Workspaces in suitable cases

Example:

```text
environments/
├── dev/
├── test/
└── prod/

modules/
├── network/
├── storage/
└── app-service/
```

---

## Scenario 7: Secret needed by Terraform

Do not commit it to Git.

Use secure CI/CD secret handling or an identity-based approach. Remember that a value used by Terraform can still end up in state depending on how the provider/resource models it.

---

## Scenario 8: CI plan succeeds but apply later differs

Infrastructure, data sources, provider behavior or configuration may have changed between plan and apply.

A safer workflow is:

```bash
terraform plan -out=tfplan
```

then, after appropriate approval:

```bash
terraform apply tfplan
```

Protect the saved plan artifact because it can contain sensitive data.

---

## Scenario 9: Resource renamed in Terraform code

Changing:

```hcl
resource "azurerm_resource_group" "old" {}
```

to:

```hcl
resource "azurerm_resource_group" "new" {}
```

changes the Terraform address.

Use a `moved` block when appropriate:

```hcl
moved {
  from = azurerm_resource_group.old
  to   = azurerm_resource_group.new
}
```

This tells Terraform the address changed rather than necessarily treating it as a new unrelated object.

---

## Scenario 10: One resource must wait for another

First prefer references:

```hcl
resource_group_name = azurerm_resource_group.example.name
```

If a real dependency exists but cannot be inferred:

```hcl
depends_on = [
  azurerm_resource_group.example
]
```

---

# 42. Rapid-Fire Interview Q&A

### What is Terraform?

An Infrastructure as Code tool used to declaratively provision/manage infrastructure through provider APIs.

### What is HCL?

HashiCorp Configuration Language, commonly used for Terraform configuration.

### What does `terraform init` do?

Initializes backend/provider/module dependencies for the working directory.

### `fmt`?

Formats Terraform configuration.

### `validate`?

Checks configuration validity/internal consistency.

### `plan`?

Shows proposed infrastructure changes.

### `apply`?

Executes approved changes.

### `destroy`?

Plans/applies destruction of managed resources for the configuration.

### What is state?

Terraform's stored mapping and metadata connecting configuration to managed infrastructure.

### Why remote state?

Team sharing, centralized protection, durability, locking and CI/CD use.

### What is state locking?

Prevents concurrent state-writing operations.

### What is a provider?

A plugin that implements resource/data-source interactions with an API/platform.

### What is a resource?

A managed infrastructure object declaration.

### What is a data source?

A way to read information rather than manage that object's lifecycle.

### Variable vs local?

Variable = external module input. Local = internal reusable/calculated value.

### What is an output?

A value exposed by a module/root configuration.

### What is a module?

A reusable collection of Terraform configuration.

### `count` vs `for_each`?

`count` uses numeric indexes; `for_each` uses stable keys from a map/set.

### What is `depends_on`?

An explicit dependency declaration.

### What is drift?

Difference between declared/known desired infrastructure and real infrastructure caused by out-of-band changes or other factors.

### What is `.terraform.lock.hcl`?

The dependency lock file recording provider selections/checksums.

### Does `sensitive = true` encrypt a secret?

No. It mainly prevents normal display in Terraform output; sensitive data may still be stored in state.

### Why should state be protected?

It contains infrastructure metadata and can contain sensitive values.

### What does import do?

Associates an existing real object with a Terraform resource address/state.

### Does import automatically write your complete resource configuration?

No. Configuration still needs to represent the desired managed object.

### What are workspaces?

Named state instances for the same configuration.

### What is `prevent_destroy`?

A lifecycle rule that rejects plans that would destroy a protected resource while that rule applies.

---

# 43. Last-Minute Cheat Sheet

## Core workflow

```bash
terraform version
terraform init
terraform fmt
terraform fmt -check
terraform validate
terraform plan
terraform plan -out=tfplan
terraform show tfplan
terraform apply tfplan
terraform destroy
```

## Variables

```bash
terraform plan -var="environment=dev"
terraform plan -var-file="dev.tfvars"
```

## Outputs

```bash
terraform output
terraform output -json
```

## State

```bash
terraform state list
terraform state show ADDRESS
terraform state mv OLD NEW
terraform state rm ADDRESS
```

## Import

```bash
terraform import ADDRESS RESOURCE_ID
```

## Workspaces

```bash
terraform workspace list
terraform workspace new dev
terraform workspace select dev
terraform workspace show
```

## Replacement

```bash
terraform plan -replace="ADDRESS"
terraform apply -replace="ADDRESS"
```

## Drift-oriented refresh plan

```bash
terraform plan -refresh-only
```

## Useful troubleshooting

```bash
terraform providers
terraform state list
terraform workspace show
terraform plan
```

---

# Interview Troubleshooting Framework

When asked:

> "Terraform deployment failed. What will you do?"

Do not answer only:

> "I will rerun the pipeline."

Use this flow:

```text
Read exact Terraform error
          |
          v
Which stage failed?
init / validate / plan / apply
          |
          v
Check configuration & variables
          |
          v
Check authentication/RBAC
          |
          v
Check backend/state/lock
          |
          v
Check provider/API error
          |
          v
Review plan and dependencies
          |
          v
Fix root cause
          |
          v
Run plan again
          |
          v
Review
          |
          v
Apply through controlled process
```

## Strong Interview Answer Example

If asked:

> "Terraform wants to destroy a production resource. What will you do?"

A strong answer:

> I would not apply the plan. I would first identify why Terraform thinks replacement or destruction is required. I would verify that I am using the correct backend, state and environment, inspect the resource in state, check recent configuration/provider changes, and look for address or `for_each`/`count` changes. If the code was refactored, I would consider a `moved` block or controlled state migration rather than recreating the resource. Only after the plan shows the intended change would I proceed through the normal approval process.

---

# Beginner Mental Model

Remember Terraform like this:

```text
CODE
 |
 | What do I WANT?
 v
Terraform Configuration
 |
 | What do I KNOW/MANAGE?
 v
Terraform State
 |
 | What actually EXISTS?
 v
Azure / Cloud
```

And remember the command lifecycle:

```text
init
 |
 v
fmt
 |
 v
validate
 |
 v
plan
 |
 v
REVIEW
 |
 v
apply
```

For interviews, understanding **why Terraform needs state, how remote state/locking work, how Terraform determines dependencies, how you review a plan, and how you safely troubleshoot unexpected changes** is more important than memorizing every command.
