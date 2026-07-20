## 10. Dependabot

**What it is:** GitHub's built-in tool that automatically scans your dependencies for known vulnerabilities and opens pull requests to update them.

>Create the config file:
```bash
mkdir -p .github
```
```bash
touch .github/dependabot.yml
```

>Example content (Go modules + GitHub Actions, checked weekly):
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

>Manually trigger a Dependabot check via the CLI:
```bash
gh api repos/:owner/:repo/dependabot/updates -X POST
```

>List currently open Dependabot alerts:
```bash
gh api repos/:owner/:repo/dependabot/alerts
```
