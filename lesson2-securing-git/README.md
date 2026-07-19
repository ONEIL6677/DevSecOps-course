# Repository Security Practices — Examples &amp; Commands

## Contents

1. [.gitignore](#1-gitignore)
2. [Native Git Pre-Commit Hooks (Custom Scripts)](#2-native-git-pre-commit-hooks-custom-scripts)
3. [Block Commits with Gitleaks](#3-block-commits-with-gitleaks)
4. [Gitleaks → Repository & History Scanning](#4-gitleaks--repository--history-scanning)
5. [Gitleaks in GitHub Actions](#5-gitleaks-in-github-actions)
6. [Branch Protection Rules](#6-branch-protection-rules)
7. [RBAC](#7-rbac)
8. [Mandatory Reviews](#8-mandatory-reviews)
9. [CODEOWNERS](#9-codeowners)
10. [Dependabot](#10-dependabot)

---

## 1. .gitignore

**What it is:** A file that tells Git which files or folders to never track — preventing secrets, build artifacts, and local config from accidentally being committed.

**Example:** A `.gitignore` for a Go project.
```
.env
*.log
/bin/
/dist/
node_modules/
.vscode/
*.pem
*.key
```

Create the file:
```bash
touch .gitignore
```

Check whether a file is currently ignored:
```bash
git check-ignore -v .env
```

If a file was already committed before being added to `.gitignore`, remove it from tracking (without deleting it locally):
```bash
git rm --cached .env
```

---

## 2. Native Git Pre-Commit Hooks (Custom Scripts)

**What it is:** Scripts that run automatically before a commit is finalized, letting you block bad commits locally before they ever reach GitHub.

**Example:** A pre-commit hook that blocks commits containing the word `TODO-SECURITY`.

Navigate to your repo's hooks folder:
```bash
cd .git/hooks
```

Create the hook file:
```bash
touch pre-commit
```

Make it executable:
```bash
chmod +x pre-commit
```

Example script content (place inside `.git/hooks/pre-commit`):
```bash
#!/bin/sh
echo "🔍 Running native pre-commit hook..."

# Block commits containing a security TODO marker
if git diff --cached | grep -q "TODO-SECURITY"; then
  echo "❌ Commit blocked: found TODO-SECURITY marker in staged changes."
  exit 1
fi

# Block commits that look like they contain hardcoded passwords, tokens, or secrets.
MATCHES=$(git diff --cached -U0 | grep -E '^\+' | grep -Ein '(password|passwd|pwd|token|secret|api[_-]?key)\s*[:=]\s*["'\''a-zA-Z0-9]')

if [ -n "$MATCHES" ]; then
  echo "❌ Commit blocked: possible hardcoded password, token, or secret found in staged changes:"
  echo "$MATCHES"
  exit 1
fi

echo "✅ Commit passed security checks."
exit 0
```

>Test it by staging a file `test` containing that marker `secret`, `TODO-SECURITY` and attempting to commit:

```bash
echo " i am want to test this file that contains pwd TODO-SECURITY to see practically if it will get commited or get the commit blocked" > test
```
>stage test file
```bash
git add test
```
> commit the staged file
```bash
git commit -m "test commit"
```
- this commit should fail
---

## 3. Block Commits with Gitleaks

**What it is:** Using Gitleaks as a pre-commit hook to automatically scan staged changes for secrets (API keys, tokens, passwords) and block the commit if any are found.
rm -r .git/hooks/pre-commit
>Install Gitleaks:
```bash
sudo apt update
```
```bash
sudo apt install gitleaks
```
>check version
```bash
gitleaks version
```
>Run it against staged changes only (ideal for a pre-commit hook):
```bash
gitleaks protect --staged --verbose
```

>Wire it into your Git hook (`.git/hooks/pre-commit`):
>create a file inside your `.git/hooks` folder called `pre-commit` and paste the code bellow inside
```bash
#!/bin/sh
gitleaks protect --staged --verbose
```

>Make the hook pre-commit file executable:
```bash
chmod +x .git/hooks/pre-commit
```
>Test it by staging a file `test` containing that marker `TODO-SECURITY`, `secret` and attempting to commit:
```bash
echo " i am want to test this file that contains secrets TODO-SECURITY secrets to see practically if it will get commited or get the commit blocked" > test
```
>stage test file
```bash
git add test
```
> commit the staged file
```bash
git commit -m "test commit"
```
>this commit should fail

---

## 4. Gitleaks → Repository & History Scanning

**What it is:** Beyond blocking new commits, scanning the entire repo (including past commit history) to catch secrets that were already committed previously.

>Scan the full repository, including history:
```bash
gitleaks detect --source . --verbose
```

>Scan and export results to a report file:
```bash
gitleaks detect --source . --report-path gitleaks-report.json
```

>Scan only a specific commit range:
```bash
gitleaks detect --source . --log-opts="--since=2024-01-01"
```

---

## 5. Gitleaks in GitHub Actions

**What it is:** Running Gitleaks automatically on every push or pull request, so leaked secrets are caught in CI even if a local hook was skipped or bypassed.

Example workflow file (`.github/workflows/gitleaks.yml`):
```yaml
name: Gitleaks Secret Scan

on: [push, pull_request]

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Run Gitleaks
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

>Trigger it manually from the CLI (if using `gh`):
```bash
gh workflow run gitleaks.yml
```

---

## 6. Branch Protection Rules

**What it is:** GitHub settings that prevent direct pushes to important branches (like `main`), requiring changes to go through pull requests with checks passing first.

>Enable branch protection on `main` using the GitHub CLI:
```bash
gh api repos/:owner/:repo/branches/main/protection \
  -X PUT \
  -F required_status_checks='{"strict":true,"contexts":["build","code-quality"]}' \
  -F enforce_admins=true \
  -F required_pull_request_reviews='{"required_approving_review_count":1}' \
  -F restrictions=null
```

>Check current protection status on a branch:
```bash
gh api repos/:owner/:repo/branches/main/protection
```

---

## 7. RBAC (Role-Based Access Control)

**What it is:** Restricting who can do what in a repository or organization e.g. only certain people can merge to `main`, push directly, or manage settings based on assigned roles.

>List current collaborators and their permission levels:
```bash
gh api repos/:owner/:repo/collaborators
```

>Add a collaborator with a specific role (e.g. `push`, `pull`, `maintain`, `admin`):
```bash
gh api repos/:owner/:repo/collaborators/past-github-USERNAMEhere -X PUT -F permission=push
```

>Create a team with a defined role at the organization level:
```bash
gh api orgs/:org/teams -X POST -f name="backend-team" -f privacy="closed"
```

>Assign that team a specific permission level on a repo:
```bash
gh api orgs/:org/teams/backend-team/repos/:owner/:repo -X PUT -f permission=push
```

---

## 8. Mandatory Reviews

**What it is:** Requiring at least one (or more) approvals from other contributors before a pull request can be merged — preventing unreviewed code from reaching production.

>Enable required reviews as part of branch protection:
```bash
gh api repos/:owner/:repo/branches/main/protection/required_pull_request_reviews \
  -X PATCH \
  -F required_approving_review_count=2 \
  -F dismiss_stale_reviews=true
```

>Check the current review requirement on a branch:
```bash
gh api repos/:owner/:repo/branches/main/protection/required_pull_request_reviews
```

---

## 9. CODEOWNERS

**What it is:** A file that automatically assigns specific people or teams as required reviewers for changes to certain files or folders — ensuring the right experts review the right code.

>Create the file at the root of the repo (or in `.github/`):
```bash
touch .github/CODEOWNERS
```

>Example content:
```
# Default owners for everything in the repo
*                   @oneil-kimbi

# Infrastructure-specific ownership
/helm/              @platform-team
/k8s/                @platform-team
/.github/workflows/  @devops-team

# Backend code
/src/                @backend-team
```

Verify GitHub recognizes it correctly by checking a pull request's "Reviewers" section — CODEOWNERS entries get auto-requested once the file is merged to the default branch.

---

## 10. Dependabot

**What it is:** GitHub's built-in tool that automatically scans your dependencies for known vulnerabilities and opens pull requests to update them.

Create the config file:
```bash
mkdir -p .github
```
```bash
touch .github/dependabot.yml
```

Example content (Go modules + GitHub Actions, checked weekly):
```yaml
version: 2
updates:
  - package-ecosystem: "gomod"
    directory: "/"
    schedule:
      interval: "weekly"

  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
```

Manually trigger a Dependabot check via the CLI:
```bash
gh api repos/:owner/:repo/dependabot/updates -X POST
```

List currently open Dependabot alerts:
```bash
gh api repos/:owner/:repo/dependabot/alerts
```

---

## Summary Table

| # | Practice | Purpose |
|---|---|---|
| 1 | .gitignore | Prevent secrets/artifacts from being tracked |
| 2 | Pre-Commit Hooks | Run custom checks before a commit is made |
| 3 | Gitleaks (Block Commits) | Stop secrets from being committed locally |
| 4 | Gitleaks (History Scan) | Find secrets already committed in the past |
| 5 | Gitleaks in GitHub Actions | Catch leaked secrets in CI, as a backstop |
| 6 | Branch Protection Rules | Prevent direct pushes to critical branches |
| 7 | RBAC | Control who can do what in the repo/org |
| 8 | Mandatory Reviews | Require human approval before merging |
| 9 | CODEOWNERS | Auto-assign the right reviewers per file/folder |
| 10 | Dependabot | Automatically patch vulnerable dependencies |