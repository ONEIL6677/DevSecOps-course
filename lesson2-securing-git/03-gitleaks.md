 ## Block Commits with Gitleaks
 **What it is:** Using Gitleaks as a pre-commit hook to automatically scan staged changes for secrets (API keys, tokens, passwords) and block the commit if any are found.
rm -r .git/hooks/pre-commit
 
 1. See What You Currently Have

>Your existing manual script lives here — this is the file about to be replaced:
```bash
cat .git/hooks/pre-commit
```

2. Install the `pre-commit` Tool

>On Ubuntu/Debian, `pip install pre-commit` will fail with an `externally-managed-environment` error (PEP 668 protection). Use `pipx` instead:
```bash
sudo apt install pipx
```
```bash
pipx ensurepath
```
```bash
pipx install pre-commit
```
>Restart your terminal (or reload your shell config), then confirm it's installed:
```bash
source ~/.bashrc
```
```bash
pre-commit --version
```

3. Create the Config File

>This file tells `pre-commit` which tools to run in this case, Gitleaks:
>command bellow creates .pre-commit-config.yaml and paste code inside
```bash
cat > .pre-commit-config.yaml << 'EOF'
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.30.0
    hooks:
      - id: gitleaks
EOF
```

In plain terms, this says: *"Before every commit, fetch the Gitleaks tool (version v8.30.0) and run its secret-scanning check."*

4. Stage the Config File

- `pre-commit` requires this file to be tracked by Git before it will run if you skip this, you'll hit:

[ERROR] Your pre-commit configuration is unstaged.
`git add .pre-commit-config.yaml` to fix this.


Fix it by staging the file:
```bash
git add .pre-commit-config.yaml
```

5. Install the Hook (This Does the Actual Replacement)

This is the key command. It overwrites whatever is currently in `.git/hooks/pre-commit` with `pre-commit`'s own managed hook:
```bash
pre-commit install
```

Expected output:

pre-commit installed at .git/hooks/pre-commit


6. Confirm the Replacement Happened

```bash
cat .git/hooks/pre-commit
```

- Your old custom script is now **gone** this file is completely rewritten by `pre-commit`. It will look nothing like the original; instead it contains `pre-commit`'s own bootstrapping code that runs whatever tools are listed in `.pre-commit-config.yaml`.

> ⚠️ **Important:** `pre-commit install` **replaces** the hook file outright it does not merge with or preserve your old script. Any custom checks (like a `TODO-SECURITY` marker or password/token regex) are no longer active unless re-added (see the note at the bottom).

7. Test It

```bash
echo 'password = "hunter2"' > test.txt
```
```bash
git add test.txt
```
```bash
git commit -m "adding secrets"
```
❌ Commit blocked.

- Gitleaks should run automatically and block the commit if it detects a real secret pattern.

8. Commit the Config File

Since it was already staged in Step 4, include it in a real commit so your team gets the same setup automatically when they clone the repo:
```bash
git commit -m "Replace manual pre-commit hook with pre-commit + Gitleaks"
```

9. (Optional) Keep Gitleaks Up to Date

>Rather than manually editing the `rev:` line in `.pre-commit-config.yaml`, update it automatically:
```bash
pre-commit autoupdate
```

---

Keeping Your Custom Checks Alongside Gitleaks

You don't have to lose your original `TODO-SECURITY` or password-pattern checks. `pre-commit` supports running multiple hooks including your own local script in the same config, alongside Gitleaks. This requires adding a second entry under `repos:` in `.pre-commit-config.yaml` pointing to a local script instead of a GitHub repo.