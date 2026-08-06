Document the process

You are a senior technical writer with AWS, Terraform, IAM, and GitHub Actions experience.

Document our centralized AWS Secrets Manager process implemented in the `shared-services` GitHub repository.

Write it as a simple end-to-end story for developers, SREs, security engineers, and repository administrators.

Keep it easy to understand, but clearly explain the security guardrails and IAM role model.

# Problem

We have more than 15 GitHub repositories.

The same secret is sometimes stored in multiple repositories under different names.

Examples:

```text
METRIC_API_KEY
METRICAPIKEY
METRICS_KEY
```

This causes:

* Duplicate secret values
* Inconsistent naming
* Difficult rotation
* Limited ownership information
* Difficulty identifying consumers
* Risk of missing repositories when a value changes
* Unclear repository permissions

# New process

Developers request secrets through the central repository:

```text
shared-services
```

They:

1. Create a branch.
2. Add one YAML request.
3. Select either specific repositories or all current repositories.
4. Open a pull request.
5. Pass automated guardrails.
6. Receive SRE or platform approval.
7. Merge the request.
8. Allow Terraform to create the AWS secret and IAM access.

The actual secret value is added directly in AWS Secrets Manager by an authorized person.

The value must never be included in GitHub or Terraform.

# Branch convention

Use:

```text
secret/<environment>/<secret-name>
```

Example:

```text
secret/prod/metrics-api-key
```

# Request examples

## Selected repositories

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

## All current repositories

```yaml
secret_name: shared-monitoring-token

environment: prod

purpose: Used by approved monitoring workflows

owner:
  team: platform-sre
  contact: platform-sre@myorg.com

access:
  scope: all
  repositories: []
```

Explain that `all` means all current approved repositories when Terraform runs.

New repositories do not automatically receive access unless the request is reapplied or updated.

# AWS naming

Terraform generates:

```text
/myorg/<environment>/<secret-name>
```

Example:

```text
/myorg/prod/metrics-api-key
```

The developer provides only:

```yaml
secret_name: metrics-api-key
environment: prod
```

# IAM role-management explanation

This section is important.

Explain that the system creates or reuses one IAM role per:

```text
repository + environment
```

Example:

```text
myorg/metrics-service + prod
    ↓
gha-metrics-service-prod-secrets-ro
```

Do not create one IAM role per secret.

If the repository uses three production secrets, the same role receives access to those three approved secret ARNs.

Example:

```text
gha-metrics-service-prod-secrets-ro
    ├── /myorg/prod/metrics-api-key
    ├── /myorg/prod/slack-webhook
    └── /myorg/prod/vendor-token
```

Explain why:

* Fewer IAM roles
* Easier administration
* Clear repository ownership
* Separate development and production access
* Easier auditing
* Smaller blast radius than one organization-wide role

# Selected repository behavior

For:

```yaml
scope: selected
```

Terraform creates or reuses roles only for the listed repositories.

Example:

```text
myorg/metrics-service
    ↓
gha-metrics-service-prod-secrets-ro

myorg/reporting-service
    ↓
gha-reporting-service-prod-secrets-ro
```

Each role receives access only to the approved secret.

# All repository behavior

For:

```yaml
scope: all
```

The automation reads the current repository list from the configured GitHub organization.

It creates or reuses a separate role for every current repository.

Do not describe this as one global role.

Use:

```text
repo-a → gha-repo-a-prod-secrets-ro
repo-b → gha-repo-b-prod-secrets-ro
repo-c → gha-repo-c-prod-secrets-ro
```

Explain that separate roles preserve repository isolation.

# IAM trust guardrail

Explain that each role trusts GitHub OIDC only for the approved repository and environment.

Example:

```text
gha-metrics-service-prod-secrets-ro
    ↓ trusted caller
myorg/metrics-service using GitHub environment prod
```

Another repository cannot access the secret by copying the role ARN.

Do not use one wildcard trust policy for all repositories.

# IAM permission guardrail

Explain that the role may call only:

```text
secretsmanager:GetSecretValue
secretsmanager:DescribeSecret
```

Only for approved secrets.

Do not allow:

```text
secretsmanager:*
Resource: *
```

# Validation guardrails

Document that pull-request validation rejects:

* Invalid branch names
* Invalid secret names
* Invalid environments
* Missing purpose
* Missing owner or contact
* Repositories outside the organization
* Repositories that do not exist
* Archived repositories
* Duplicate repository entries
* Duplicate secret requests
* Wildcard repository names
* Plaintext secret values
* Ambiguous combinations of `scope: all` and manually listed repositories
* Terraform plans that attempt unexpected deletion

# Production guardrails

Explain that production requests require:

* SRE or platform approval
* Protected GitHub environment approval
* Terraform plan before apply
* Terraform apply only after merge
* OIDC authentication
* No direct push to the default branch

# Secret-value guardrail

Clearly emphasize:

Terraform creates the AWS Secrets Manager secret container only.

Terraform must not create:

```text
aws_secretsmanager_secret_version
```

The value is added separately by an authorized person.

Never place secret values in:

* YAML
* Git commits
* Pull requests
* GitHub issues
* Terraform variables
* Terraform state
* Terraform outputs
* Workflow logs

# End-to-end developer story

Use this example:

The Reporting team needs a production metrics API key.

The developer:

1. Creates:

```text
secret/prod/metrics-api-key
```

2. Adds:

```text
secret-requests/prod-metrics-api-key.yaml
```

3. Describes the purpose.
4. Adds the owner team and contact.
5. Selects the repositories.
6. Opens a pull request.
7. Fixes validation errors.
8. Receives SRE approval.
9. Merges the request.
10. Terraform creates:

```text
/myorg/prod/metrics-api-key
```

11. Terraform creates or reuses:

```text
gha-metrics-service-prod-secrets-ro
gha-reporting-service-prod-secrets-ro
```

12. Each role receives access only to the new secret.
13. An authorized person adds the value in AWS Secrets Manager.
14. The repositories retrieve it through OIDC.

# Benefits

Explain:

* One secret naming convention
* One request process
* One AWS value to rotate
* No duplicated GitHub secret copies
* Clear owner and contact
* Clear repository access
* One role per repository and environment
* Least-privilege IAM permissions
* Better auditing
* Safer production access
* Easier revocation
* Easier migration and cleanup

# Troubleshooting

Include:

* Branch validation failed
* Secret name is invalid
* Repository is outside the organization
* Repository does not exist
* `scope: selected` has no repositories
* `scope: all` also contains repository names
* Terraform created the secret but it has no value
* Repository receives `AccessDenied`
* Repository is not included in the request
* Role exists but does not include the new secret
* Production apply is waiting for approval

# Document structure

Use:

1. Overview
2. Current Problem
3. New Process
4. Developer Request
5. Naming Convention
6. Selected vs All Repository Access
7. IAM Role Management
8. IAM Trust and Permission Guardrails
9. Pull-Request Guardrails
10. What Happens After Merge
11. Adding the Secret Value
12. Using the Secret
13. Secret Rotation
14. Removing Access
15. Security Rules
16. Advantages
17. Troubleshooting
18. End-to-End Example

Keep the documentation readable in approximately ten minutes.

