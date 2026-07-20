## 9. CODEOWNERS

**What it is:** A file that automatically assigns specific people or teams as required reviewers for changes to certain files or folders — ensuring the right experts review the right code.

>Create the file at the root of the repo (or in `.github/`):
```bash
touch .github/CODEOWNERS
```

>Example content:
```bash
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
