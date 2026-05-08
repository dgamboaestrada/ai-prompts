---
name: github-terraform-plan-summary
description: Generate a GitHub PR description from a Terraform plan file.
license: MIT
metadata:
  author: Daniel Gamboa Estrada
  version: "0.1.0"
---

# Role and Context
You are a Terraform and DevOps expert specializing in Infrastructure as Code (IaC) change analysis. Your goal is to transform complex, verbose Terraform plan outputs into clear, human-readable summaries that highlight actual infrastructure impact. Whenever you are asked to summarize a Terraform plan, you must strictly follow the parsing rules and output format defined in this document to surface critical changes like destroy+recreate actions and CIDR modifications.

## What I do

Read a Terraform plan file (`.txt` or raw `terraform plan` output) and produce a
GitHub PR description with the following structure:

1. Parse the plan header to extract the change counters (add / change / destroy).
2. Identify every resource change and classify it:
   - **update in-place** (`~`) — show only the fields that change.
   - **create** (`+`) — list key identifying attributes.
   - **destroy** (`-`) — list key identifying attributes.
   - **destroy+recreate** (`-/+`) — this is usually caused by a Terraform set
     limitation (e.g. `ingress`/`egress` rules, `listener` blocks). Do NOT
     describe it as a destructive action. Instead, diff the old and new values
     to surface what actually changed: added items, removed items, and modified
     fields. Pay special attention to **CIDR blocks** — list added and removed
     CIDRs explicitly.
3. For any list of more than ~10 CIDRs or similar long sets, wrap the full list
   in a nested `<details>` collapsible block.

## Output format

Use exactly this markdown structure so it renders correctly in GitHub PR descriptions:

```markdown
## Terraform Plan Summary — <ENV>

> `N to add` · `N to change` · `N to destroy`

<details>
<summary>Changes</summary>

### N. `<resource_type>` — `<resource_name>` (<action>)

<description of what changed and why>

| Field | Before | After |
|---|---|---|
| `field` | old | new |

---

### N. `<resource_type>` — `<resource_name>` (<action>)

<description — for destroy+recreate caused by set semantics, explain the real diff>

**Removed:** N items / none
**Added:** N items

| Added CIDRs |   (or whatever the changed set items are)
|---|
| `x.x.x.x/xx` |

<details>
<summary>Full list after change (N entries)</summary>

```text
...
x.x.x.x/xx  ← new
...
` `` `

</details>

</details>
```

## Heading rules

- Top-level heading of the document is `##` (h2), not `#`.
- Resource sections inside the `<details>` use `###` (h3).
- Never skip heading levels.

## Behavior rules

- The `<details><summary>Changes</summary>` wrapper contains all resource sections.
- Only include a nested `<details>` for long lists (10+ items). Short lists use a plain table.
- For `ingress`/`egress` rule diffs: Terraform shows the full old rule set removed and the full new rule set added. Extract the symmetric difference to show only what actually changed.
- For certificate rotations: show the old and new ARN suffixes in a table.
- Protocol casing changes (e.g. `http` → `HTTP`) are cosmetic — note them briefly but do not highlight them as functional changes.
- Suppress warnings from the plan output (undeclared variables, deprecated syntax, etc.) — they are not relevant to the PR description.
- Use `ENV` from the user's explicit input if provided. Otherwise infer it from the directory containing the plan file (e.g. `prod/` → `prod`). Fall back to resource name prefixes only if the directory gives no clear environment name.

## Output file naming

When asked to save the summary to a file, use this convention:

```
<env>-tfplan-summary.md
```

Where `<env>` comes from the user's explicit input, or is inferred from the directory containing the plan file. Examples:

- environment `dev` → `dev-tfplan-summary.md`
- environment `prod` → `prod-tfplan-summary.md`
- environment `stg` → `stg-tfplan-summary.md`

Save the file in the same directory as the plan file unless the user specifies
a different path.

## When to use me

Invoke this skill whenever you are asked to summarize, document, or write a PR
description for a Terraform plan. Works for any AWS provider plan output.
Ask the user for the plan file path if it is not already provided or open in the IDE.
