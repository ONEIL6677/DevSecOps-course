# Gitleaks — Scanning Automatically on GitHub (GitHub Actions)

## What This Is For

Your pre-commit hook only protects commits made **on your own machine**. If someone forgets to install the hook, bypasses it (`--no-verify`), or commits from a different computer, a secret could still slip through. Running Gitleaks in **GitHub Actions** acts as a backstop — it automatically scans every push and pull request on GitHub's servers, no matter where the commit came from.

---

## Option A: Set It Up Using the Command Line (git)

1. Create the Workflow Folder

```bash
mkdir -p .github/workflows
```

2. Create the Workflow File

```bash
cat > .github/workflows/gitleaks.yml << 'EOF'
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
EOF
```

3. Commit and Push It

```bash
git add .github/workflows/gitleaks.yml
```
```bash
git commit -m "Add Gitleaks GitHub Actions workflow"
```
```bash
git push
```

That's it — from now on, this scan runs automatically on every push and every pull request.

---

## Option B: Set It Up Manually Using the GitHub Website (No Terminal)

If you'd rather do this entirely through the browser instead of the command line:

1. Go to Your Repository on GitHub

Open your repository's page on **github.com**.

2. Open the Actions Tab

Click **Actions** in the top navigation bar of the repository.

3. Start a New Workflow

- If this is your first workflow, click **"set up a workflow yourself"** (a small blue link, usually below the suggested templates).
- If you already have other workflows, click **New workflow**, then look for **"set up a workflow yourself"** near the top of the templates page instead of picking a suggested one.

4. Name the File

GitHub opens a new file editor with a default name like `main.yml`. Click on that filename at the top and rename it to:
```
gitleaks.yml
```
Make sure it's still located inside `.github/workflows/` (GitHub does this automatically for you).

5. Paste the Workflow Content

Delete any placeholder text GitHub added, and paste this in:
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

6. Commit the File

Scroll down to the bottom of the page. You'll see a **"Commit changes"** box:
- Add a short commit message, e.g. `Add Gitleaks GitHub Actions workflow`
- Choose **"Commit directly to the `main` branch"** (or create a new branch and open a pull request, if your repo requires that)
- Click the green **Commit changes** button

7. Watch It Run

Go back to the **Actions** tab you should see "Gitleaks Secret Scan" listed as a workflow, and a run in progress (or already completed) for the commit you just made.

---

### How to View the Results

1. Go to the **Actions** tab.
2. Click on the most recent **"Gitleaks Secret Scan"** run.
3. Click the **scan** job to expand it.
4. Look at the **"Run Gitleaks"** step:
   - ✅ Green checkmark = no secrets found.
   - ❌ Red X = a secret was detected — click the step to expand the log and see exactly which file and line triggered it.

---

### Optional Trigger It Manually Without a New Commit

If you have the GitHub CLI installed:
```bash
gh workflow run gitleaks.yml
```