## Gitleaks — Scanning Your Whole Repository & History

### What This Is For
Your pre-commit hook only checks **new** commits going forward. It doesn't look at code that was already committed in the past. This guide covers scanning your **entire repository history** to catch secrets that may have been committed before you set up any protection.

> ⚠️ **Note:** The old command `gitleaks detect` is outdated. Use `gitleaks git` instead — it does the same job.

---

1. Make Sure You're in Your Project Folder

```bash
cd /path/to/your-repo
```

2. Run a Full Scan of the Repository (Including History)

>This checks every commit ever made in the repo, not just the latest one:
```bash
gitleaks git --source . --verbose
```

**What you'll see:**
- If nothing is found → a clean summary, no secrets detected.
- If something is found → details on the secret, which file, and which commit it's in.

3. Save the Results to a File (Optional but Recommended)

>Instead of just reading results in the terminal, save them so you can review or share the list later:
```bash
gitleaks git --source . --report-path gitleaks-report.json
```

This creates a file called `gitleaks-report.json` in your current folder, listing every finding.

4. Scan Only a Specific Time Range (Optional)

>If your repo has a long history and a full scan is slow, you can limit it to commits made after a certain date:
```bash
gitleaks git --source . --log-opts="--since=2024-01-01"
```

Change `2024-01-01` to whatever start date you want.

---

### Quick Summary

| Goal | Command |
|---|---|
| Scan everything | `gitleaks git --source . --verbose` |
| Scan and save a report | `gitleaks git --source . --report-path gitleaks-report.json` |
| Scan only recent history | `gitleaks git --source . --log-opts="--since=2024-01-01"` |

### What to Do If It Finds a Secret

1. **Rotate/change the secret immediately** assume it's compromised, since it was exposed in Git history.
2. Remove it from the current code and replace it with an environment variable or a secrets manager (e.g. HashiCorp Vault).
3. Optionally, remove it from Git history entirely using a tool like `git filter-repo` note this rewrites history and requires everyone on the team to re-clone.