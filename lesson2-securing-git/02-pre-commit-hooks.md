## 2. Native Git Pre-Commit Hooks (Custom Scripts)

**What it is:** Scripts that run automatically before a commit is finalized, letting you block bad commits locally before they ever reach GitHub.

**Example:** A pre-commit hook that blocks commits containing the word `TODO-SECURITY`.

>Navigate to your repo's hooks folder:
```bash
cd .git/hooks
```

>Create the hook file:
```bash
touch pre-commit
```

>Make it executable:
```bash
chmod +x pre-commit
```

>Example script content (place inside `.git/hooks/pre-commit`):
```bash
#!/bin/sh

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