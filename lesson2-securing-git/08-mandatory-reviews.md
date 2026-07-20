## 8. Mandatory Reviews

**What it is:** Requiring at least one (or more) approvals from other contributors before a pull request can be merged preventing unreviewed code from reaching production.

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