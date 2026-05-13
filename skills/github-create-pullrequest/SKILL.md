---
name: github-create-pullrequest
description: Create a pull request using GitHub CLI with a structured, high-quality description.
license: MIT
metadata:
  author: Daniel Gamboa Estrada
  version: "0.2.0"
---

# Role and Context

You are a GitHub expert specializing in Pull Request creation using the GitHub CLI (`gh`). Whenever asked to create a PR, follow this workflow strictly to ensure well-documented, properly pushed PRs.

## Execution workflow

### Step 1 — Detect base branch and repository status

```bash
BASE_BRANCH=$(git branch -r | grep -E 'origin/(main|master)$' | sed 's/.*origin\///' | sort | head -n 1)
echo "Base branch: $BASE_BRANCH"
git status
git log --oneline -10
```

### Step 2 — Analyze changes for the description

```bash
# Commits ahead of base
git log --oneline origin/$BASE_BRANCH..HEAD

# Changed files
git diff origin/$BASE_BRANCH...HEAD --name-status

# Commit details (adjust N to number of commits in branch)
git show --stat HEAD~N..HEAD
```

### Step 3 — Push the branch

Always push before creating the PR:

```bash
git push origin <branch-name>
```

### Step 4 — Write PR body to temp file

Build the description and write it to a temp file to avoid shell escaping issues:

```bash
cat > /tmp/pr_body.md << 'EOF'
## Description

[Brief 1-2 line summary of the PR purpose]

### Main Changes

1. **[Change category]**
   - Specific detail

### Modified Files

- `path/to/file.ext`
EOF
```

### Step 5 — Preview the body

Display the temp file and ask the user for confirmation before creating the PR:

```bash
cat /tmp/pr_body.md
```

### Step 6 — Create the PR

```bash
# Normal PR (default)
gh pr create --base $BASE_BRANCH --head <branch-name> \
  --title "<branch-name>: <description>" \
  --body "$(cat /tmp/pr_body.md)"

# Draft PR (only if explicitly requested by the user)
gh pr create --draft --base $BASE_BRANCH --head <branch-name> \
  --title "<branch-name>: <description>" \
  --body "$(cat /tmp/pr_body.md)"
```

**Title rules:**
- Format: `<branch-name>: <description>`
- Respect the exact casing of the branch name
  - `TASK-1234` → `TASK-1234: ...`
  - `feature-auth` → `feature-auth: ...`

### Step 7 — Open in browser

```bash
gh pr view --web
```

---

## PR body structure

```markdown
## Description

[Brief 1-2 line summary of the PR purpose]

### Main Changes

1. **[Change category]**
   - Specific detail
   - Additional detail if applicable

2. **[Change category]**
   - Specific detail

### Modified Files

- `path/to/file1.ext`
- `path/to/file2.ext`
```

---

## Common errors

| Error | Cause | Solution |
|-------|-------|----------|
| `No commits between $BASE_BRANCH and <branch>` | Branch not pushed or wrong base | Run `git push origin <branch>` and verify `$BASE_BRANCH` |
| `Head sha can't be blank` | Branch doesn't exist in remote | Push the branch first |
| Title prefix wrong casing | Not matching branch name exactly | Verify with `git branch --show-current` |
