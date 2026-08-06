You are a senior DevOps, AWS security, Terraform, and GitHub Actions engineer.

Build a simple centralized secrets-management solution in our existing GitHub repository:

`shared-services`

Keep the implementation simple, secure, and easy for developers to use.

Do not introduce:

* Databases
* Web applications
* Lambda functions
* Custom APIs
* Secrets copied into GitHub repository secrets
* Secret values stored in Terraform
* One IAM role per secret

Use:

* One YAML request file
* GitHub pull requests
* GitHub Actions
* Terraform
* AWS Secrets Manager
* GitHub OIDC
* One IAM role per repository and environment

# Goal

A developer should be able to:

1. Create a branch using a standard naming convention.
2. Add one YAML secret-request file.
3. Specify the secret purpose, environment, owner, contact, and repositories that need access.
4. Open a pull request.
5. Pass automated guardrail validation.
6. Receive SRE or platform-team approval.
7. Merge the pull request.
8. Allow Terraform to create the AWS secret and repository-specific IAM permissions automatically.

Terraform creates the AWS Secrets Manager secret resource, but it must not create or store the secret value.

An authorized person will add the value directly in AWS Secrets Manager after the resource is created.

# Developer branch naming

Require:

```text
secret/<environment>/<secret-name>
```

Examples:

```text
secret/dev/metrics-api-key
secret/stage/slack-webhook
secret/prod/vendor-token
```

Allowed environments:

```text
dev
test
stage
prod
```

Guardrails:

* Branch must start with `secret/`
* Environment must be one of the allowed values
* Secret name must use lowercase letters, numbers, and hyphens
* No underscores, spaces, uppercase letters, or special characters
* Branch secret name must match the request-file secret name

# Secret request file

Developers add one file under:

```text
secret-requests/
```

File format:

```text
<environment>-<secret-name>.yaml
```

Example:

```text
secret-requests/prod-metrics-api-key.yaml
```

Use this request structure:

```yaml
secret_name: metrics-api-key

environment: prod

purpose: Used by reporting workflows to publish application metrics

owner:
  team: sre-observability
  contact: sre-observability@myorg.com

access:
  scope: selected
  repositories:
    - myorg/metrics-service
    - myorg/reporting-service
```

For all repositories in the organization:

```yaml
secret_name: shared-monitoring-token

environment: prod

purpose: Shared token used by approved monitoring workflows

owner:
  team: platform-sre
  contact: platform-sre@myorg.com

access:
  scope: all
  repositories: []
```

Allowed access scopes:

```text
selected
all
```

# Access-scope behavior

## Selected repositories

When:

```yaml
access:
  scope: selected
```

The developer must provide at least one repository:

```yaml
repositories:
  - myorg/metrics-service
  - myorg/reporting-service
```

Terraform must create or reuse one IAM role for each repository and environment.

Example:

```text
myorg/metrics-service + prod
    ↓
gha-metrics-service-prod-secrets-ro
```

## All repositories

When:

```yaml
access:
  scope: all
```

The developer must not manually list repositories.

The automation should query the configured GitHub organization and expand `all` into the current approved repository list.

For every repository, Terraform must create or reuse a separate repository-and-environment IAM role.

Do not create one organization-wide role shared by every repository.

Example:

```text
myorg/repo-a
    ↓
gha-repo-a-prod-secrets-ro

myorg/repo-b
    ↓
gha-repo-b-prod-secrets-ro
```

The `all` option means all current approved repositories at the time Terraform runs.

New repositories added later must not automatically receive access unless:

* The secret request is updated and reapplied, or
* A separate controlled synchronization process is intentionally introduced later

Keep Phase 1 explicit and predictable.

# AWS secret naming

The developer provides:

```yaml
secret_name: metrics-api-key
environment: prod
```

Terraform generates:

```text
/myorg/prod/metrics-api-key
```

Use:

```text
/<organization>/<environment>/<secret-name>
```

The organization name must be configured once as a Terraform variable.

Developers must not manually provide the full AWS secret path.

# AWS tags

Automatically add:

```text
Name
Environment
Purpose
OwnerTeam
Contact
ManagedBy
AccessScope
```

Example:

```text
Name        = metrics-api-key
Environment = prod
Purpose     = Used by reporting workflows to publish application metrics
OwnerTeam   = sre-observability
Contact     = sre-observability@myorg.com
ManagedBy   = shared-services
AccessScope = selected
```

Do not allow developers to provide arbitrary AWS tags in Phase 1.

This prevents users from overriding required governance tags.

# IAM role-management model

This requirement is important.

Use one IAM role per:

```text
GitHub repository + AWS environment
```

Examples:

```text
gha-metrics-service-dev-secrets-ro
gha-metrics-service-prod-secrets-ro
gha-reporting-service-prod-secrets-ro
```

Do not create:

```text
One role per secret
One unrestricted role for the entire organization
One role with access to all Secrets Manager resources
```

## Role reuse

If a repository already has a role for the same environment, reuse it.

Example:

```text
gha-metrics-service-prod-secrets-ro
```

If the repository is approved for three production secrets, the same role should receive permission to read those three secrets.

Terraform should aggregate approved secret ARNs for each repository and environment.

Example:

```text
gha-metrics-service-prod-secrets-ro
    ├── /myorg/prod/metrics-api-key
    ├── /myorg/prod/slack-webhook
    └── /myorg/prod/vendor-token
```

Do not create three separate IAM roles.

## IAM trust policy

Each IAM role must trust GitHub OIDC.

The trust policy must restrict access to:

* The configured GitHub organization
* One specific repository
* The configured GitHub environment when possible
* The expected AWS audience

Another repository must not be able to assume the role by copying its ARN.

Prefer a GitHub OIDC subject similar to:

```text
repo:myorg/metrics-service:environment:prod
```

Do not use a trust policy such as:

```text
repo:myorg/*
```

for repository-specific roles.

Do not trust all repositories through one wildcard role.

## IAM permissions policy

The role should receive only:

```text
secretsmanager:GetSecretValue
secretsmanager:DescribeSecret
```

Only for the secrets approved for that repository.

Never generate:

```json
{
  "Action": "secretsmanager:*",
  "Resource": "*"
}
```

Add `kms:Decrypt` only when a customer-managed KMS key is intentionally configured.

Restrict `kms:Decrypt` to the specific KMS key.

## IAM role outputs

Terraform should output a repository-to-role mapping:

```text
myorg/metrics-service:
  environment: prod
  role: gha-metrics-service-prod-secrets-ro
  secrets:
    - /myorg/prod/metrics-api-key
```

This mapping should also appear in the GitHub Actions summary.

# Guardrails

Implement the following guardrails.

## Request-content guardrails

Reject any request containing fields such as:

```text
value
secret_value
password
credential
token_value
api_key_value
private_key
client_secret
```

Reject requests containing suspicious long encoded strings or likely credentials.

Do not print suspected secret content in the validation error.

Use an error such as:

```text
Potential secret value detected. Remove all credential values from the request.
```

## Repository guardrails

Every repository must:

* Use the format `<organization>/<repository>`
* Belong to the configured GitHub organization
* Exist in GitHub
* Not be archived
* Not appear more than once
* Not contain wildcard characters
* Not use a different organization

Reject:

```text
another-company/repository
myorg/*
*
```

## Access-scope guardrails

For:

```yaml
scope: selected
```

Require one or more repositories.

For:

```yaml
scope: all
```

Require an empty or omitted repository list.

Reject ambiguous combinations such as:

```yaml
access:
  scope: all
  repositories:
    - myorg/repo-a
```

## Production guardrails

For `environment: prod`:

* Require SRE or platform-team approval
* Require a protected GitHub environment for Terraform apply
* Require manual approval before Terraform apply
* Require the owner contact
* Require a meaningful purpose
* Do not allow direct pushes to the default branch
* Do not allow Terraform apply from pull-request workflows

## Terraform guardrails

The pull-request workflow may run:

```text
terraform fmt -check
terraform validate
terraform plan
```

It must never run:

```text
terraform apply
```

Terraform apply must only run:

* After merge into the default branch
* From the trusted `shared-services` workflow
* Through GitHub OIDC
* Using a protected GitHub environment
* Using an approved AWS provisioning role

Use a remote Terraform backend with state locking.

Do not store Terraform state in Git.

Do not place secret values in:

* Variables
* Outputs
* State
* Plan output
* Workflow logs

Do not create:

```hcl
aws_secretsmanager_secret_version
```

in Phase 1.

## Duplicate guardrails

Reject:

* Duplicate AWS secret paths
* Duplicate request filenames
* Duplicate logical combinations of environment and secret name
* Duplicate repository entries
* Two request files trying to manage the same AWS secret
* Conflicting owners for the same secret

## Destructive-change guardrails

Do not automatically delete an AWS secret merely because a YAML request file was removed.

In Phase 1:

* Prevent accidental secret deletion
* Use lifecycle protection where appropriate
* Fail the plan when a managed request disappears unexpectedly
* Require a future explicit retirement process for deletion

Do not implement automatic deletion as part of the initial solution.

## Workflow-permission guardrails

Use minimum GitHub Actions permissions.

Validation workflow:

```yaml
permissions:
  contents: read
  pull-requests: read
```

Deployment workflow:

```yaml
permissions:
  contents: read
  id-token: write
```

Add other permissions only when they are required.

Do not use:

```yaml
permissions: write-all
```

## Third-party action guardrails

Use official or organization-approved GitHub Actions.

Pin actions to approved versions or commit SHAs according to organization policy.

Do not use unverified marketplace actions.

# Pull-request validation workflow

Create:

```text
.github/workflows/validate-secret-request.yml
```

Run when:

```text
secret-requests/**
terraform/**
scripts/**
```

change in a pull request.

Validate:

* Branch name
* Request filename
* YAML syntax
* Required fields
* Allowed environment
* Secret-name format
* Purpose is present and meaningful
* Owner team is present
* Owner contact is present
* Access scope is valid
* Repository list matches the selected scope
* Repositories belong to the organization
* Repositories exist
* Repositories are not archived
* No duplicate repositories
* No duplicate secret
* No plaintext secret value
* Terraform formatting
* Terraform validation
* Terraform plan

Provide clear developer-friendly errors.

# Deployment workflow

Create:

```text
.github/workflows/deploy-secret-request.yml
```

Run only after changes are merged into the default branch.

The workflow should:

1. Validate all requests again.
2. Authenticate to AWS using GitHub OIDC.
3. Run Terraform plan.
4. Require protected-environment approval for production.
5. Run Terraform apply.
6. Generate a GitHub Actions summary.
7. Generate or update a Markdown inventory.

Example summary:

```text
Secret created:
  /myorg/prod/metrics-api-key

Purpose:
  Used by reporting workflows to publish application metrics

Owner:
  sre-observability

Contact:
  sre-observability@myorg.com

Access scope:
  selected

Repository roles:
  myorg/metrics-service
    gha-metrics-service-prod-secrets-ro

  myorg/reporting-service
    gha-reporting-service-prod-secrets-ro

Secret value:
  Not managed by Terraform.
  Add the value directly in AWS Secrets Manager.
```

# Repository structure

Keep the repository structure simple:

```text
shared-services/
├── secret-requests/
│   └── prod-metrics-api-key.yaml
├── scripts/
│   └── validate_secret_request.py
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── providers.tf
├── reports/
│   └── secret-inventory.md
├── examples/
│   ├── selected-repositories.yaml
│   ├── all-repositories.yaml
│   └── consuming-workflow.yml
├── .github/
│   ├── CODEOWNERS
│   ├── pull_request_template.md
│   └── workflows/
│       ├── validate-secret-request.yml
│       └── deploy-secret-request.yml
└── README.md
```

Do not add unnecessary folders.

# CODEOWNERS

Require SRE or platform approval:

```text
/secret-requests/ @myorg/sre-team
/terraform/ @myorg/platform-team
/.github/workflows/ @myorg/platform-team
```

Use placeholders where exact team names are unknown.

# Consuming repository example

Provide a GitHub Actions example that:

1. Requests `id-token: write`.
2. Uses a GitHub environment matching the requested AWS environment.
3. Assumes the repository-and-environment IAM role.
4. Retrieves the approved AWS secret.
5. Exposes it as an environment variable.
6. Uses the environment variable in later steps.

Example:

```text
AWS secret:
/myorg/prod/metrics-api-key

Repository role:
gha-metrics-service-prod-secrets-ro

Workflow environment variable:
METRICS_API_KEY
```

# Expected developer experience

```text
Create branch
    ↓
Add one YAML request
    ↓
Choose selected repositories or all repositories
    ↓
Open pull request
    ↓
Guardrail validation runs
    ↓
SRE or platform team approves
    ↓
Merge
    ↓
Terraform creates AWS secret
    ↓
Terraform creates or reuses one role per repository and environment
    ↓
Terraform grants each role access only to approved secrets
    ↓
Authorized person adds the secret value in AWS
```

# Deliverables

Generate:

1. Architecture summary
2. Assumptions
3. Repository structure
4. Example request for selected repositories
5. Example request for all repositories
6. Validation script
7. Terraform implementation
8. IAM trust-policy implementation
9. IAM permission-policy implementation
10. IAM role aggregation logic
11. Pull-request validation workflow
12. Deployment workflow
13. CODEOWNERS
14. Pull-request template
15. Inventory report
16. Consuming workflow example
17. README
18. Local testing commands
19. Guardrail test cases

Before writing code, explain:

1. How repository access is expanded
2. How IAM roles are grouped
3. How roles are reused
4. How secret permissions are aggregated
5. How production approvals work
6. How accidental deletion is prevented

Keep Phase 1 simple, readable, secure, and production-oriented.
