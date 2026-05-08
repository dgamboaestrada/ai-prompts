---
name: github-create-pullrequest
description: Create a pull request using GitHub CLI.
license: MIT
metadata:
  author: Daniel Gamboa Estrada
  version: "0.1.0"
---

# Role and Context
You are a GitHub expert specializing in efficient Pull Request management using the GitHub CLI (`gh`). Your goal is to guide the user through a seamless PR creation workflow, ensuring all changes are properly pushed, analyzed, and documented. Whenever you are asked to create a pull request, you must strictly follow the steps defined in this document to maintain high-quality descriptions and repository integrity.

## Full Workflow

### 1. Verify Repository Status
Before creating a PR, always verify:
```bash
git status
git log --oneline -10
```

### 2. Push the Branch
**CRITICAL**: Always push the branch before attempting to create the PR:
```bash
git push origin <BRANCH-NAME>
```

### 3. Analyze Changes for the PR Body
Before creating the PR, analyze the changes to write a high-quality description.

#### 3.1 View branch commits
```bash
git log --oneline origin/main..HEAD
```
This shows the commits that are in your branch but not in main.

#### 3.2 View changed files
```bash
git diff origin/main...HEAD --name-status
```
Shows which files were modified (M), added (A), or deleted (D).

#### 3.3 View commit details
```bash
git show --stat HEAD~N..HEAD
```
Replace N with the number of commits you want to review. For example, `HEAD~3..HEAD` for the last 3 commits.

#### 3.4 Recommended Body Structure

The body must follow this markdown structure:

```markdown
## Description

[Brief 1-2 line summary of the PR purpose]

### Main Changes

1. **[Change category 1]**
   - Specific detail
   - Additional detail if applicable

2. **[Change category 2]**
   - Specific detail

### Modified Files

- `path/to/file1.ext`
- `path/to/file2.ext`
```

**Full Body Example:**
```markdown
## Description

This PR refactors the authentication middleware to improve security and reduce code duplication.

### Main Changes

1. **Consolidated authentication logic**
   - Merged duplicate auth checks into a single middleware
   - Added support for JWT and OAuth strategies

2. **Enhanced security measures**
   - Implemented rate limiting for failed auth attempts
   - Added request signature validation

### Modified Files

- `src/middleware/auth.ts`
- `src/utils/security.ts`
- `tests/auth.test.ts`
```

### 4. Create the Pull Request
Use the `gh pr create` command with these considerations:

**By default**: Create the PR as a normal PR (not draft). Only use `--draft` if the user explicitly requests it.

```bash
# Normal PR (default)
gh pr create --base main --head <BRANCH-NAME> --title "<PREFIX>: <DESCRIPTION>" --body "<DETAILED-DESCRIPTION>"

# Draft PR (only if requested) - --draft flag at the beginning for better visibility
gh pr create --draft --base main --head <BRANCH-NAME> --title "<PREFIX>: <DESCRIPTION>" --body "<DETAILED-DESCRIPTION>"
```

#### Important Title Rules:
- **Format**: `<branch-name>: <title>`
- **Case Sensitivity**: Respect EXACTLY the casing of the branch name
  - If the branch is `TASK-1234` → use `TASK-1234: ...`
  - If the branch is `feature-123` → use `feature-123: ...`
  - If the branch is `Fix-Bug-456` → use `Fix-Bug-456: ...`

#### Full Example:
```bash
# For branch TASK-1234 (Normal PR)
gh pr create --base main --head TASK-1234 --title "TASK-1234: Add user authentication middleware" --body "Added authentication middleware to protect routes that require user login. Includes JWT token validation and automatic redirection to login page for unauthenticated users."

# For branch feature-auth-fix (Normal PR)
gh pr create --base main --head feature-auth-fix --title "feature-auth-fix: Fix authentication timeout issue" --body "Fixed authentication timeout by increasing session duration and adding retry logic for failed auth requests."

# For branch TASK-1234 (Draft PR - only if requested)
gh pr create --draft --base main --head TASK-1234 --title "TASK-1234: Add user authentication middleware" --body "Added authentication middleware to protect routes that require user login. Includes JWT token validation and automatic redirection to login page for unauthenticated users."
```

### 5. Open the PR in the Browser
Once the PR is created, open it immediately in the browser:
```bash
gh pr view --web
```

## Common Errors and Solutions

### Error: "No commits between main and <branch>"
**Cause**: Branch hasn't been pushed
**Solution**: Run `git push origin <branch-name>` before creating the PR

### Error: "Head sha can't be blank"
**Cause**: Branch doesn't exist in the remote repository
**Solution**: Push the branch first

### Title Format Error
**Cause**: Not respecting the branch name casing
**Solution**: Verify the exact branch name with `git branch` and use exactly the same format

## Verification Checklist

Before creating a PR, verify:
- [ ] Changes are committed locally
- [ ] Commits have been reviewed with `git log`
- [ ] Modified files have been analyzed with `git diff`
- [ ] Branch has been pushed to the remote repository
- [ ] Branch name in `--head` matches the real name exactly (including casing)
- [ ] Title prefix uses exactly the same format as the branch name
- [ ] Body includes: Description, Main Changes, and Modified Files
- [ ] Body description is clear, structured, and explains what and why

## Sequence of Commands (Full Example)

```bash
# 1. Verify status and commits
git status
git log --oneline -10

# 2. Analyze changes for the body
git log --oneline origin/main..HEAD
git diff origin/main...HEAD --name-status
git show --stat HEAD~3..HEAD

# 3. Push (if not already done)
git push origin TASK-1234

# 4. Create PR with structured description (Normal PR by default)
gh pr create --base main --head TASK-1234 \
  --title "TASK-1234: Description of changes" \
  --body "## Description

Brief summary of changes.

### Main Changes

1. **Category 1**
   - Change detail
   - Additional detail

2. **Category 2**
   - Change detail

### Modified Files

- \`src/file1.ts\`
- \`src/file2.ts\`"

# 4b. Create Draft PR (only if explicitly requested) - --draft flag at the beginning
gh pr create --draft --base main --head TASK-1234 \
  --title "TASK-1234: Description of changes" \
  --body "## Description

Brief summary of changes.

### Main Changes

1. **Category 1**
   - Change detail
   - Additional detail

2. **Category 2**
   - Change detail

### Modified Files

- \`src/file1.ts\`
- \`src/file2.ts\`"

# 5. Open in browser
gh pr view --web
```

## Important Notes

- **Always** push before creating the PR
- **By default** create normal PRs (not draft). Only use `--draft` if the user explicitly requests it
- **Analyze** changes before writing the body (use `git log` and `git diff`)
- **Respect** exactly the casing of the branch name
- **Structure** the body with recommended sections (Description, Main Changes, Modified Files)
- **Be specific** in the description - explain what changed and why
- **Open** the PR in the browser for final review
- The `gh pr view --web` command works from any repository directory after creating the PR
- A good description facilitates code review and project documentation
- Draft PRs are useful for work in progress that is not yet ready for review
