Choose Option 1 — dedicated Git repo, but local Git only for now. Do not configure or create any remote repository. Keep main container itself as a plain directory. The new dev-ai-workspace should use local Git history to protect the AI workspace during this migration. Continue assessment only for now; do not create it until I approve the migration.

You are helping me redesign an existing multi-repository DevOps workspace into a maintainable, governed, AI-assisted engineering workspace.

This workspace has grown organically over time while I used GitHub Copilot and other AI tools. There are useful files already present, but the organization is inconsistent. Before making any changes, I want you to fully understand what exists, preserve it, classify it, and propose a cleaner architecture.

This first run is **assessment and planning only**.

Do NOT reorganize files yet.

Do NOT create, move, rename, merge, archive, or delete anything unless I explicitly approve it after reviewing your proposal.

---

# 1. Background

This workspace contains:

* Multiple Git repositories from the same organization.
* A hub-and-spoke architecture where different framework repositories depend on shared-services repositories.
* Shared APIs.
* Reusable GitHub Actions workflows.
* Infrastructure-as-code.
* Application/platform/framework repositories.
* Downloaded Confluence/wiki documentation.
* Existing operational runbooks.
* Jira/work-item analysis generated during previous engineering tasks.
* Existing `.github/instructions`, `.github/knowledge`, `.github/runbooks`, and other AI-generated or AI-maintained files.
* GitHub Copilot-specific instructions.
* Potential future Claude Code configuration.
* MCP integrations that may provide read-only access to:

  * AWS
  * Azure
  * GCP
  * Git
  * filesystem
  * other engineering systems

Historically, when I worked on Jira tickets, I would provide the ticket description to Copilot.

Copilot then created Markdown files, often under `.github`, for:

* ticket analysis
* trackers
* findings
* implementation notes
* architecture investigation

Separately, I also asked AI to understand the different repositories and how they interact.

The AI reviewed:

* source code
* GitHub Actions workflows
* shared-services repositories
* downloaded Confluence/wiki content

and created knowledge files.

I also created runbooks by combining:

* existing Confluence documentation
* existing source code
* GitHub workflows
* runtime investigation using MCP/cloud access

These files are useful, but they are now mixed together and difficult to govern.

---

# 2. Overall Goal

Design a long-term AI workspace architecture that supports:

1. Durable engineering knowledge.
2. Jira/work-item-specific analysis.
3. Human-readable runbooks.
4. Reusable Agent Skills.
5. Specialized subagents.
6. Hooks and deterministic guardrails.
7. MCP-based investigation.
8. Consistent file-generation rules.
9. Knowledge provenance and traceability.
10. Claude Code.
11. GitHub Copilot.
12. Portability between AI tools where practical.
13. Controlled self-improvement of the AI setup.
14. Reduced context pollution in long AI sessions.
15. Safe operation across multiple repositories and cloud environments.

The result should eventually feel like a governed AI engineering assistant rather than a collection of random prompts and Markdown files.

---

# 3. Core Mental Model

Keep these concepts separate.

## Instructions

Permanent behavioral rules for the AI.

Examples:

* where files should be created
* security rules
* repository conventions
* engineering standards
* when approval is required
* how knowledge should be validated

Instructions should not contain ticket-specific analysis.

---

## Knowledge

Durable facts about the environment.

Examples:

* organization architecture
* repository relationships
* hub-and-spoke design
* shared services
* authentication patterns
* cloud architecture
* deployment architecture
* CI/CD architecture
* terminology
* known dependencies

Knowledge should be curated and relatively stable.

---

## Skills

Reusable procedures or workflows.

Examples:

* Jira ticket analysis
* framework analysis
* incident triage
* runbook generation
* repository investigation
* deployment investigation

Think of a Skill as a reusable runbook for the AI.

---

## Subagents

Specialized workers with focused context and limited tools.

Examples:

* repo explorer
* cloud investigator
* ticket analyst
* implementation engineer
* documentation reviewer

Use subagents where isolating investigation prevents unnecessary context from polluting the parent session.

---

## Hooks

Deterministic automation and policy enforcement.

Examples:

* repository freshness checks
* secret scanning
* dangerous-command protection
* file-location enforcement
* formatting
* validation

A hook should be used when something should happen reliably rather than depending on the model remembering an instruction.

---

## Runbooks

Human-readable operational procedures.

Examples:

* deploy checkout service
* investigate credential rotation
* rollback deployment
* validate cloud configuration

Runbooks are primarily for engineers, although AI may also use them as references.

---

## Work artifacts

Temporary or ticket-specific investigation material.

Examples:

* Jira ticket analysis
* implementation plan
* findings
* investigation notes
* temporary trackers

These should not be stored as permanent AI instructions.

---

## MCP

Controlled access to external systems and runtime information.

Examples:

* AWS
* GCP
* Azure
* Git
* filesystem
* monitoring platforms
* other engineering APIs

Prefer read-only access for investigation unless explicit modification is required.

---

# 4. Phase 0 — Preserve the Current Workspace

Before proposing or performing any reorganization, preserve a complete record of the current AI-related workspace.

## 4.1 Git safety assessment

Inspect:

* Git repositories present in this workspace
* current branches
* working-tree status
* uncommitted changes
* staged changes
* untracked files
* ignored files that appear AI-related
* repositories that are not currently clean

Do NOT run destructive Git operations.

Do NOT run:

* `git reset`
* `git clean`
* forced checkout
* forced branch switching
* force push
* commands that discard working-tree changes

Do not assume that everything important is already committed.

Explicitly identify AI-related files that are:

* committed
* modified
* untracked
* ignored

Explain what would and would not be protected by Git.

---

# 5. Backup Strategy

Recommend a safe way to preserve the exact pre-migration state.

Prefer a combination of:

* existing Git history
* backup branch or tag
* inventory
* archive strategy

Propose an appropriate backup branch/tag name such as:

`backup/pre-ai-workspace-reorg-YYYY-MM-DD`

Do NOT create the branch or tag during this assessment unless I approve it.

If multiple repositories are involved, explain whether each repository needs its own backup reference.

If there are untracked files, explicitly explain how those should be preserved before reorganization.

---

# 6. Complete AI File Inventory

Identify existing AI-related files throughout the workspace.

Do not limit inspection to `.github`.

Look for:

* `.github/instructions/`
* `.github/knowledge/`
* `.github/runbooks/`
* `.github/snyk/`
* `.github/copilot-instructions.md`
* existing agent files
* Claude-related files
* Copilot-related files
* Jira ticket files
* analysis files
* tracker files
* temporary notes
* architecture documentation
* generated runbooks
* AI-generated knowledge
* instruction files
* prompt files
* scripts created specifically for AI workflows
* any other likely AI-generated or AI-maintained content

For every relevant file, classify it.

Produce a table:

| Current path | Purpose | Classification | Proposed destination | Proposed action | Confidence |
| ------------ | ------- | -------------- | -------------------- | --------------- | ---------- |

Use classifications such as:

* AI instruction
* durable knowledge
* Jira/work-item artifact
* runbook
* reusable procedure
* generated analysis
* tracker
* temporary content
* vendor-specific configuration
* security tooling/configuration
* unclear

Use proposed actions:

* KEEP
* MOVE
* MERGE
* RENAME
* ARCHIVE
* REVIEW
* EVENTUAL DELETE

Do not actually perform these actions yet.

---

# 7. Preserve Migration History

Recommend governance files for tracking the migration.

Prefer something similar to:

```text
ai-governance/
├── migration/
│   ├── pre-migration-inventory.md
│   ├── migration-map.md
│   └── cleanup-candidates.md
│
├── proposals/
│
└── archive/
    └── legacy-ai-files/
```

Explain whether this structure is appropriate or recommend a better equivalent.

The purpose of each should be:

## `pre-migration-inventory.md`

Record every relevant AI-related file that existed before migration.

## `migration-map.md`

Track:

* original path
* new path
* action performed
* date
* reason

## `cleanup-candidates.md`

Track legacy files that might eventually be deleted after the new architecture has been proven.

## `archive/legacy-ai-files/`

Temporary preservation of obsolete AI-generated files when deletion would otherwise lose traceability.

---

# 8. Archive Before Delete

During future migration phases:

Do NOT immediately delete old AI-generated files.

Prefer:

1. KEEP temporarily, or
2. MOVE to a legacy archive.

Only recommend deletion after:

* the content has been migrated
* duplication has been confirmed
* the new structure has been used successfully
* provenance has been preserved

No existing artifact should disappear without being represented in the migration history.

---

# 9. Consolidation Rules

If multiple existing files contain overlapping information, do not simply choose one.

For each proposed consolidation, identify:

* all source files
* destination canonical document
* information that should be retained
* duplicate content
* conflicting information
* obsolete information
* anything intentionally excluded
* reason for consolidation

Do not silently discard information.

---

# 10. Proposed Portable Core

Design the workspace so that as much as practical is independent of Claude or Copilot.

Evaluate a portable structure similar to:

```text
workspace/
│
├── AGENTS.md
├── CLAUDE.md
│
├── knowledge/
├── runbooks/
├── work/
├── sources/
├── scripts/
├── ai-governance/
│
├── .claude/
│
├── .github/
│
└── repositories/
```

Do not assume these exact names are automatically correct.

Inspect the current workspace first and recommend the best structure.

The portable core should preferably include things such as:

* `knowledge/`
* `runbooks/`
* `work/`
* `sources/`
* reusable scripts
* engineering conventions
* Agent Skills where portability is supported

Vendor-specific adapters should contain only what needs to be vendor-specific.

---

# 11. Claude-Specific Architecture

Evaluate appropriate use of:

* `CLAUDE.md`
* `.claude/agents/`
* `.claude/settings.json`
* `.claude/skills/`
* Claude Code hooks
* permissions
* MCP integrations

Keep `CLAUDE.md` concise.

Treat it as a constitution, not an encyclopedia.

It should eventually describe broad rules such as:

* where work artifacts belong
* where durable knowledge belongs
* security expectations
* when subagents should be used
* read-only cloud investigation preference
* documentation/provenance requirements
* approval requirements

Do not place large amounts of architecture documentation into `CLAUDE.md`.

---

# 12. GitHub Copilot Architecture

Evaluate appropriate use of:

* `.github/copilot-instructions.md`
* `.github/instructions/`
* `.github/agents/`
* Agent Skills supported by GitHub Copilot
* Copilot hooks where supported

Do not assume that Claude and Copilot configuration formats are interchangeable.

Prefer shared principles with thin vendor-specific adapters.

---

# 13. Authoritative References

Before recommending implementation details, consult current official documentation.

Do not rely on remembered or outdated behavior for AI-tool configuration.

## Claude Code

Use official Anthropic/Claude documentation for:

* Claude Code overview
* project instructions / `CLAUDE.md`
* Skills
* Subagents
* Hooks
* Permissions
* Settings
* MCP
* supported file locations

## Agent Skills

Use:

`https://agentskills.io`

as the authority for the open Agent Skills specification.

Review:

* `SKILL.md`
* directory structure
* metadata
* progressive disclosure
* references
* scripts
* skill triggering
* evaluation guidance

## GitHub Copilot

Use official GitHub documentation for:

* repository custom instructions
* path-specific instructions
* Agent Skills
* custom agents
* Copilot CLI/agent capabilities
* hooks
* supported configuration locations

Clearly identify capabilities that are:

* portable
* Claude-specific
* Copilot-specific

Do not invent portability where it does not exist.

---

# 14. Knowledge Architecture

Design a clean knowledge hierarchy.

Evaluate something similar to:

```text
knowledge/
├── START-HERE.md
├── README.md
├── glossary.md
│
├── architecture/
│   ├── organization-map.md
│   ├── hub-spoke-model.md
│   ├── shared-services.md
│   └── repo-dependencies.md
│
├── repositories/
│   ├── checkout-framework.md
│   ├── credential-framework.md
│   └── ...
│
├── cloud/
│   ├── aws.md
│   ├── azure.md
│   └── gcp.md
│
└── cicd/
    └── github-actions.md
```

Adapt this to the actual workspace.

Do not create unnecessary files.

Prefer canonical documents over many small overlapping files.

---

# 15. Knowledge Creation Rules

Agents must search existing knowledge before creating new knowledge documents.

Avoid patterns such as:

* `analysis.md`
* `analysis-new.md`
* `analysis-final.md`
* `notes2.md`
* `framework-analysis-new.md`

Prefer:

* updating the canonical document
* adding a section
* explicitly replacing outdated content
* recording provenance

If creating a new knowledge document is necessary, explain why no existing canonical document was suitable.

---

# 16. Knowledge Provenance

Every durable knowledge document should eventually include traceability.

Recommend a consistent format containing fields or sections such as:

* Purpose
* Scope
* Relevant repositories
* Relevant source paths
* Confluence/wiki references
* Runtime/MCP validation
* Last validated
* Owner, if known
* Related runbooks
* Related work items

Do not require meaningless metadata simply for completeness.

Use only fields that add useful traceability.

---

# 17. Evidence Classification

When agents produce findings, require them to distinguish:

## CONFIRMED

Directly observed in:

* source code
* configuration
* runtime system
* cloud API
* authoritative system output

## DOCUMENTED

Found in existing documentation but not independently verified.

## INFERRED

A conclusion derived from available evidence.

## UNKNOWN

Unable to determine confidently.

Never present inferred information as confirmed fact.

---

# 18. Jira / Work-Item Structure

Ticket-specific work should not live under permanent AI instruction directories.

Evaluate a structure such as:

```text
work/
└── TLCSEEIS-3613/
    ├── ticket.md
    ├── analysis.md
    ├── plan.md
    ├── findings.md
    └── implementation-notes.md
```

Do not require every file for every ticket.

Use only what is needed.

Recommend lifecycle rules for:

* active work
* completed work
* archived work
* durable knowledge extracted from completed work

Explain when findings should be promoted into `knowledge/` or `runbooks/`.

---

# 19. Work-to-Knowledge Promotion

Do not automatically turn every ticket finding into permanent knowledge.

Design a controlled workflow such as:

```text
Work Item
   ↓
Investigation
   ↓
Findings
   ↓
Candidate Knowledge Update
   ↓
Review
   ↓
Knowledge / Runbook
```

Require evidence for durable knowledge.

Avoid automatically rewriting architecture documentation after every ticket.

---

# 20. Initial Skills

Evaluate whether these are appropriate initial Agent Skills:

## `jira-work-item`

Purpose:

Analyze newly assigned Jira work.

Potential responsibilities:

* parse ticket
* identify requirements
* identify affected repositories
* inspect existing knowledge
* inspect runbooks
* delegate repository investigation
* delegate cloud investigation if needed
* produce consistent work artifacts

Expected output location:

`work/<ticket-id>/`

---

## `framework-analysis`

Purpose:

Understand how a framework or platform works.

Potential sources:

* repository code
* reusable workflows
* shared-services dependencies
* existing knowledge
* downloaded Confluence content
* runtime investigation where needed

Expected durable output:

canonical document under `knowledge/`.

---

## `runbook-builder`

Purpose:

Create or improve operational runbooks using evidence from:

* source code
* deployment workflows
* existing documentation
* runtime/cloud investigation

Expected output:

`runbooks/`

---

## `incident-triage`

Purpose:

Investigate operational failures without unnecessarily modifying systems.

Potential investigation areas:

* repository
* CI/CD
* cloud
* deployment
* configuration
* logs

Prefer read-only investigation.

---

For every recommended skill specify:

* purpose
* triggering scenarios
* inputs
* workflow
* allowed tools
* subagents
* expected output
* file locations
* failure behavior
* safety requirements
* references
* what should NOT be included in the skill

Keep the initial number of skills small.

Do not create skills for one-off tasks.

---

# 21. Initial Subagents

Evaluate these roles.

Do not assume all are required.

## `repo-explorer`

Potential responsibilities:

* trace code
* search repository relationships
* inspect GitHub Actions
* identify APIs/libraries/shared services
* trace dependencies
* return concise evidence to parent agent

Default behavior:

read-only

Should not modify source code.

---

## `cloud-investigator`

Potential responsibilities:

* AWS investigation
* Azure investigation
* GCP investigation
* MCP queries
* resource relationships
* IAM/authentication investigation
* runtime configuration inspection

Prefer read-only credentials.

Should not change cloud resources by default.

---

## `ticket-analyst`

Potential responsibilities:

* understand Jira requirement
* identify acceptance criteria
* identify ambiguities
* identify impacted systems
* separate facts from assumptions

---

## `implementation-engineer`

Potential responsibilities:

* modify approved code
* implement approved plans
* run tests
* validate changes

This should be one of the few agents permitted to modify source code.

---

## `documentation-reviewer`

Potential responsibilities:

* compare findings with existing knowledge
* identify duplicate documents
* identify stale runbooks
* recommend knowledge promotion
* verify provenance

---

For each subagent recommend:

* role
* triggering conditions
* tools
* denied tools
* writable paths
* read-only paths
* expected response to parent
* whether its context should remain isolated
* whether it needs any skills
* whether MCP is permitted

Prefer least privilege.

---

# 22. Context Management

One of the goals is to prevent one huge AI context from containing:

* dozens of repositories
* hundreds of wiki pages
* cloud outputs
* ticket analysis
* implementation output

Recommend how the parent agent should delegate focused investigations to subagents.

The parent agent should receive summarized findings and evidence rather than every raw file/tool result where possible.

Explain which activities benefit most from isolated contexts.

---

# 23. Hooks and Deterministic Guardrails

Design a small initial set of useful hooks.

Do not create hooks just because hooks exist.

Hooks should solve real deterministic problems.

Evaluate the following.

---

## Hook 1 — Repository Freshness

Before certain investigations:

* inspect Git status
* detect current branch
* detect uncommitted changes
* fetch remote changes

Prefer:

`git fetch`

Do NOT blindly run `git pull` across every repository.

If safe and explicitly allowed, consider:

`git pull --ff-only`

Never:

* discard changes
* reset
* clean
* force checkout
* force pull
* force push

If a repository is dirty, report it rather than altering it.

---

## Hook 2 — Secret Detection

Detect likely secrets before:

* writing knowledge
* writing ticket artifacts
* committing generated output
* sending content to external tools where appropriate

Examples:

* AWS access keys
* GitHub PATs
* API tokens
* bearer tokens
* private keys
* passwords
* connection strings
* cloud credentials

Prefer deterministic secret detection rather than asking an LLM to notice secrets.

Never print detected secret values.

Redact or report their location safely.

Recommend established secret-scanning tools if appropriate.

---

## Hook 3 — File Location Enforcement

Prevent agents from creating artifacts in arbitrary locations.

Examples:

Ticket artifacts must go under:

`work/<ticket-id>/`

Durable knowledge must go under:

`knowledge/`

Runbooks must go under:

`runbooks/`

Claude configuration must go under approved Claude locations.

Copilot configuration must go under approved GitHub locations.

Temporary outputs should have an approved location.

If an agent tries to create:

`.github/TLCSEEIS-3613-analysis.md`

the operation should be rejected or redirected according to a deterministic rule.

---

## Hook 4 — Dangerous Command Protection

Require explicit approval or block destructive operations such as:

* `terraform apply`
* `terraform destroy`
* `kubectl delete`
* destructive AWS CLI calls
* destructive Azure CLI calls
* destructive GCP CLI calls
* `git push --force`
* `git reset --hard`
* `git clean`
* recursive destructive file deletion

Avoid relying exclusively on pattern matching if stronger permission mechanisms exist.

Combine hooks with tool permissions and least-privilege credentials.

---

## Hook 5 — Knowledge Validation

When durable knowledge changes, validate required properties such as:

* evidence/source references
* no detected secrets
* valid location
* no obvious duplicate canonical document
* last validation metadata where applicable

---

## Hook 6 — Code Quality

After approved source-code changes, evaluate language/tool-specific validation such as:

* formatting
* linting
* tests
* Terraform formatting/validation
* YAML validation
* shell checks

Only run checks relevant to the changed files.

---

For each proposed hook specify:

* event
* action
* failure behavior
* whether it blocks or warns
* where implementation should live
* Claude support
* Copilot support
* portability

---

# 24. MCP Security Model

Recommend a safe MCP strategy.

Prefer:

* read-only MCP for investigation
* narrowly scoped access
* explicit write permissions
* separate investigation and implementation permissions where useful

Treat an AI agent similar to a junior engineer:

* access only what it needs
* prefer read
* log actions
* require approval for destructive actions
* do not expose secrets unnecessarily

Clearly separate:

* MCP access
* agent permissions
* hook enforcement
* cloud IAM permissions

Do not treat any one of these as a replacement for the others.

---

# 25. Generated File Governance

The system should never create files randomly because "that seemed convenient."

Design deterministic artifact rules.

For every common workflow specify:

* whether a file should be created
* where it should be created
* naming convention
* whether an existing file should be updated instead
* retention behavior

Examples:

### Jira analysis

`work/<ticket>/`

### Durable repository knowledge

`knowledge/repositories/`

### Architecture knowledge

`knowledge/architecture/`

### Runbooks

`runbooks/`

### Governance proposals

`ai-governance/proposals/`

### Temporary AI scratch artifacts

Recommend an appropriate controlled location or determine whether they should remain in context only.

---

# 26. Self-Improvement

I want the AI integration to improve over time, but it must not silently rewrite its own governance.

Design a controlled improvement process.

For example:

```text
Observation
   ↓
Improvement Proposal
   ↓
Human Review
   ↓
Approved Change
   ↓
Evaluation
   ↓
Adoption
```

Potential proposal topics:

* new skill
* skill modification
* subagent modification
* instruction update
* hook improvement
* missing knowledge category
* duplicate documentation
* context inefficiency

Consider:

`ai-governance/proposals/`

for proposed changes.

The AI may recommend improvements.

It must not silently modify:

* core instructions
* security guardrails
* hooks
* permissions
* skill governance
* agent privileges

without approval.

---

# 27. Evaluation Strategy

Design a simple evaluation strategy for the AI setup.

For each Skill, eventually maintain test prompts such as:

* examples that should trigger the skill
* examples that should not trigger it
* expected artifact location
* expected subagent usage
* expected safety behavior

Evaluate skills in fresh contexts where practical.

Also evaluate:

* unnecessary file creation
* unnecessary subagent invocation
* wrong skill triggering
* context growth
* source traceability
* accidental secret exposure
* unsafe tool usage

Recommend where these evaluations should live.

---

# 28. Avoid Overengineering

Do not create:

* dozens of skills
* dozens of subagents
* complicated orchestration
* hooks for everything
* excessive metadata
* unnecessary framework code

Start with the smallest architecture that solves the real problems.

Prefer approximately:

* a few core instructions
* a small number of Skills
* a small number of specialist agents
* a few high-value hooks
* clear directories

The system should be understandable to another DevOps engineer.

---

# 29. Initial Target Workflow

Evaluate whether the future system should support a workflow similar to this:

```text
User gives Jira ticket
        ↓
Main agent
        ↓
jira-work-item Skill
        ↓
Search existing knowledge/runbooks
        ↓
ticket-analyst
        ↓
repo-explorer if needed
        ↓
cloud-investigator if needed
        ↓
Main agent correlates evidence
        ↓
work/<ticket>/
        ↓
Human approves implementation
        ↓
implementation-engineer
        ↓
Tests / validation
        ↓
documentation-reviewer
        ↓
Candidate knowledge/runbook update
        ↓
Human review
```

Recommend improvements if there is a better design.

---

# 30. Required Output — Current-State Assessment

For this first run, produce the following.

## A. Executive Summary

Briefly explain:

* what is currently working well
* what is disorganized
* biggest risks
* biggest opportunities

---

## B. Current Directory Classification

Map the current workspace into:

* instructions
* knowledge
* work artifacts
* runbooks
* temporary content
* vendor-specific configuration
* source documentation
* unknown/misplaced content

---

## C. Complete Pre-Migration Inventory

Provide the inventory described earlier.

Do not omit files simply because they look old.

---

## D. Git Safety Report

Identify:

* dirty repositories
* untracked AI files
* ignored AI files
* files not currently protected by Git
* recommended backup approach

---

## E. Problems Identified

Look specifically for:

* Jira artifacts stored as instructions
* duplicate knowledge
* stale generated analysis
* temporary files treated as durable knowledge
* unclear canonical documents
* overly broad instructions
* missing provenance
* uncontrolled file creation
* context pollution
* potential secret exposure
* excessive tool permissions
* unsafe Git automation

---

## F. Proposed Target Directory Tree

Provide the complete proposed structure.

Explain each major directory.

Avoid creating the files yet.

---

## G. Migration Map

Produce:

| Current path | Classification | Recommended destination | Action | Reason |
| ------------ | -------------- | ----------------------- | ------ | ------ |

Do not execute the migration.

---

## H. Canonical Knowledge Map

Identify which existing documents should become canonical for major topics.

Identify likely duplicates that should eventually be merged.

---

## I. Proposed Initial Skills

Recommend no more than approximately 3–5 initial Skills.

Explain why each deserves to exist.

---

## J. Proposed Initial Subagents

Recommend no more than approximately 3–5 initial subagents.

Explain why each needs isolated context.

---

## K. Proposed Initial Hooks

Recommend only the highest-value hooks.

Prioritize:

* safety
* consistency
* secret protection
* Git safety

---

## L. Claude vs Copilot Portability Matrix

Produce:

| Capability | Portable | Claude-specific | Copilot-specific | Recommendation |
| ---------- | -------- | --------------- | ---------------- | -------------- |

Include:

* project instructions
* Skills
* subagents
* hooks
* permissions
* MCP
* knowledge
* runbooks
* work artifacts

---

## M. Proposed Files

List every new file or directory you recommend creating.

For each explain its purpose.

Do not create them.

---

## N. Migration Phases

Recommend an incremental implementation plan.

Prefer something similar to:

### Phase 0

Backup + inventory.

### Phase 1

Directory organization and core instructions.

### Phase 2

First Skill.

### Phase 3

First subagents.

### Phase 4

Hooks and guardrails.

### Phase 5

Copilot compatibility.

### Phase 6

Evaluation and controlled self-improvement.

Adjust if your analysis suggests a better sequence.

---

## O. Risks

Identify risks such as:

* losing useful AI-generated knowledge
* stale documentation becoming authoritative
* duplicate knowledge
* cloud permissions
* secret exposure
* accidental Git changes
* agent context explosion
* vendor lock-in
* too much automation

---

## P. Open Questions

If you cannot determine something from the workspace, list it here.

Do not guess.

---

# 31. Final Constraint

This first execution is strictly:

**INSPECT → INVENTORY → CLASSIFY → RESEARCH → RECOMMEND**

It is NOT:

**CREATE → MOVE → DELETE → IMPLEMENT**

Do not modify the workspace.

Do not reorganize files.

Do not install packages.

Do not modify Git state.

Do not modify repositories.

Do not modify cloud resources.

Do not create Claude configuration yet.

Do not create Copilot configuration yet.

Do not create Skills yet.

Do not create subagents yet.

Do not create hooks yet.

Do not archive anything yet.

Present the full assessment and migration proposal first.

Wait for my explicit approval before making any changes.
