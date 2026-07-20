
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