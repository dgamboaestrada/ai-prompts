---
name: github-terraform-publish-plan
description: Parse and publish Terraform plan output as a formatted GitHub PR comment with clean formatting, no warnings by default, and resource-focused analysis.
license: MIT
metadata:
  author: Daniel Gamboa Estrada
  version: "1.1.1"
---

# Role and Context

You are a Terraform plan parser and GitHub automation specialist. Your role is to extract key information from Terraform plan outputs and publish them as structured, formatted comments directly to GitHub pull requests.

## Execution workflow

### Step 1 — Resolve the plan file

Read the file path provided by the user. If none given, default to `tfplan.txt` in the current directory.

```bash
cat <plan-file-path>
```

### Step 2 — Detect the PR number

Auto-detect from the current branch:

```bash
gh pr list --state open --head $(git branch --show-current) --json number,title
```

If detection fails or returns no results, ask the user for the PR number explicitly.

### Step 3 — Parse the plan

Extract from the plan file:
- **Target command**: last `[INFO]` line at the bottom, e.g. `[INFO] terraform plan -var-file=env.tfvars -target='...'`
- **Module-service**: first-level module from the target (e.g. `module.myservice`, not `module.myservice.module.sub`)
- **Summary line**: pattern `Plan: X to add, Y to change, Z to destroy`
- **Resource changes**: all `# module... will be...` blocks with their diffs
- **Warnings**: strip by default; include only if the user explicitly requests them (e.g. "con warnings", "include warnings")

### Step 4 — Write comment to temp file

Build the formatted comment and write it to a temp file for preview and reuse. **Extract the COMPLETE terraform plan output from the plan file (lines 1 through the "Plan: X to add..." summary line) and include it verbatim in the Plan output section. DO NOT summarize, truncate, or edit any resource blocks:**

```bash
# Extract the complete plan (from line 1 to and including the summary line)
PLAN_LINES=$(grep -n "^Plan:" <plan-file-path> | head -1 | cut -d: -f1)
sed -n "1,${PLAN_LINES}p" <plan-file-path> > /tmp/plan_output.txt

# Write the formatted comment header
cat > /tmp/pr_comment.txt << 'EOF'
Terraform plan for <module-service>
---
Target plan: `<target-command>`
Summary: `<plan-summary>`

<details><summary>Changes</summary>

- <resource_type> (<resource_name>)
  <what changed>

</details>

<details><summary>Plan output</summary>
<p>

```hcl
EOF

# Append the COMPLETE plan output (unmodified, verbatim)
cat /tmp/plan_output.txt >> /tmp/pr_comment.txt

# Close the markdown block
cat >> /tmp/pr_comment.txt << 'EOF'
```

</p>
</details>
EOF
```

### Step 5 — Preview the comment

Display the temp file so the user can review before publishing:

```bash
cat /tmp/pr_comment.txt
```

Ask the user for explicit confirmation (e.g. "yes", "si", "proceed") before continuing.

### Step 6 — Publish to GitHub PR

```bash
gh pr comment <pr-number> --body "$(cat /tmp/pr_comment.txt)"
```

Use `cat /tmp/pr_comment.txt` to avoid shell escaping issues with backticks and special characters in the plan output.

---

## Comment template

```
Terraform plan for <module-service>
---
Target plan: `<target-command>`
Summary: `<plan-summary>`

<details><summary>Changes</summary>

- <resource_type> (<resource_name>)
  <1-2 lines describing what changed>

</details>

<details><summary>Plan output</summary>
<p>

```hcl
<complete-terraform-plan-from-line-1-to-plan-summary>
```

</p>
</details>
```

### Formatting rules

- **Changes bullets**: use `-` (no emojis) with resource type and name, 1-2 lines per resource
- **Plan output section**: **ALWAYS include the COMPLETE raw terraform plan output** from the plan file (from line 1 through the "Plan: X to add, Y to change, Z to destroy" summary line). **DO NOT summarize, truncate, or edit any resource blocks.** Include all resources, attributes, and changes exactly as they appear in the file. This ensures full transparency and traceability for reviewers. Strip warnings, deprecation notices, and variable declaration warnings (lines after the "Plan: X to add..." summary line) by default; include them only if the user explicitly requests them (e.g. "con warnings", "include warnings", "include deprecations").
- **Warnings (optional)**: only when explicitly requested, add a third collapsible section:

```markdown
<details><summary>Warnings</summary>

**Undeclared Variables:** my_variable (env.tfvars), another_variable (terraform.tfvars)
**Deprecated Syntax:** Quoted references in ignore_changes; interpolation-only expressions
**Resource Targeting:** Plan uses -target — may not represent all requested changes

</details>
```

---

## Full example

```bash
# Step 2 — detect PR
gh pr list --state open --head $(git branch --show-current) --json number,title
# → [{"number":42,"title":"feat: rotate TLS certificate for mywebapp"}]

# Step 3 — write comment to temp file with COMPLETE plan output
PLAN_FILE="path/to/tfplan.txt"
PLAN_LINES=$(grep -n "^Plan:" "$PLAN_FILE" | head -1 | cut -d: -f1)
sed -n "1,${PLAN_LINES}p" "$PLAN_FILE" > /tmp/plan_output.txt

cat > /tmp/pr_comment.txt << 'EOF'
Terraform plan for module.mywebapp
---
Target plan: `terraform plan -var-file=int.tfvars -target='module.mywebapp.aws_elb.public_elb'`
Summary: `Plan: 0 to add, 2 to change, 0 to destroy`

<details><summary>Changes</summary>

- aws_elb (mywebapp-public-elb)
  Rotating SSL certificate: `aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa` → `bbbbbbbb-bbbb-bbbb-bbbb-bbbbbbbbbbbb`
  HTTPS listener update on port 443

- aws_security_group (public_elb_sg)
  Ingress port 443: Adding 3 new CDN provider IPs
  Total: 8 CDN endpoints, 5 unchanged + 3 new additions

</details>

<details><summary>Plan output</summary>
<p>

```hcl
EOF

# Append the complete plan output (verbatim, unmodified)
cat /tmp/plan_output.txt >> /tmp/pr_comment.txt

cat >> /tmp/pr_comment.txt << 'EOF'
```

</p>
</details>
EOF

# Step 4 — preview
cat /tmp/pr_comment.txt

# Step 5 — publish (after user confirmation)
gh pr comment 42 --body "$(cat /tmp/pr_comment.txt)"
```

---

## Key principles

1. **Full transparency**: Always include the COMPLETE plan output, never summarize or truncate
2. **Verbatim extraction**: Copy the plan section from the file exactly as-is; do not edit individual resources
3. **Warnings handling**: Strip technical warnings by default for readability; include only when explicitly requested
4. **Traceability**: The complete plan ensures reviewers can audit every change without external reference
