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