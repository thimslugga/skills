---
name: git-branch-practices
description: Git branch naming conventions, best practices, and workflow guidance. Use this skill whenever the user asks about git branch names, branching strategies, branch naming conventions, how to name a branch, git workflow organization, or mentions wanting to clean up their branch naming. Also trigger when the user is setting up a new project's git workflow, writing contributing guides, creating git hooks for branch validation, or asks anything about feature/bugfix/hotfix/release branch patterns. Trigger even for casual questions like "what should I name this branch" or "how do teams usually name branches".
---

# Git Branch Naming Best Practices

## Core Philosophy

Branch names are communication. A good branch name tells you three things at a glance: what kind of work it is, what it relates to, and roughly what changed. Everything else is noise.

Think of branch names like labeled folders in a filing cabinet. The label `feature/add-user-auth` tells you exactly which drawer to look in (feature work) and what's inside (user authentication). Compare that to a branch called `stuff` or `my-fix-2`.

## The Format

The widely adopted pattern is:

```
<type>/<description>
```

Or when using a ticket tracker:

```
<type>/<ticket-id>-<description>
```

### Branch Type Prefixes

These are the standard prefixes most teams converge on:

| Prefix       | Purpose                                    | Example                              |
|------------- |--------------------------------------------|--------------------------------------|
| `feature/`   | New functionality                          | `feature/user-auth`                  |
| `bugfix/`    | Fixing a bug (non-urgent)                  | `bugfix/login-redirect-loop`         |
| `hotfix/`    | Urgent production fix                      | `hotfix/crash-on-checkout`           |
| `release/`   | Release preparation                        | `release/v2.1.0`                     |
| `refactor/`  | Code improvement, no behavior change       | `refactor/split-monolith-routes`     |
| `docs/`      | Documentation only                         | `docs/update-api-readme`             |
| `test/`      | Adding or updating tests                   | `test/payment-edge-cases`            |
| `chore/`     | Build, deps, tooling, CI                   | `chore/upgrade-node-18`             |

You don't need all of these. Pick the ones that match your workflow and stick with them. For a solo project, `feature/`, `bugfix/`, and `hotfix/` cover 90% of cases.

### Why Slashes Over Dashes for the Separator

Slashes (`/`) between prefix and description are preferred over dashes for practical reasons:

1. **Tab completion in the shell** -- type `git checkout feat<TAB>` and your shell groups all feature branches together.
2. **Filtering** -- `git branch --list "feature/*"` shows only feature branches. Try doing that cleanly with dashes.
3. **Remote renaming** -- slashes let you remap namespaces when pushing: `git push origin 'refs/heads/feature/*:refs/heads/review/feature/*'`

Use hyphens (`-`) to separate words within the description portion: `feature/add-user-auth`, not `feature/addUserAuth` or `feature/add_user_auth`.

## The Rules

### 1. Lowercase everything

Git is case-sensitive. `Feature/Login` and `feature/login` are different branches. Mixing cases causes confusion, especially across Linux/macOS/Windows where filesystem case sensitivity differs. Just go lowercase.

### 2. Use hyphens between words

```
# good
feature/reset-password-flow

# bad
feature/reset_password_flow    # underscores are harder to type
feature/resetPasswordFlow      # camelCase doesn't scan well in terminal
feature/Reset-Password-Flow    # mixed case
```

### 3. Keep it short but meaningful

Aim for under 50 characters total. The name should convey the intent without reading like a sentence.

```
# good
feature/user-auth
bugfix/fix-header-overlap

# too long
feature/user-profile-interface-update-for-enhanced-user-experience-based-on-feedback

# too vague
feature/update
bugfix/fix
```

### 4. Include ticket/issue IDs when you have them

If your team uses Jira, GitHub Issues, Linear, etc., embed the ID. It creates a direct link between the code and the tracking system.

```
feature/PROJ-123-user-auth
bugfix/GH-456-login-redirect
hotfix/TICK-789-null-pointer
```

### 5. No bare numbers

A branch named `8712` or `123` tells you nothing. Always include context alongside the number.

### 6. Avoid personal names as the sole identifier

`john/stuff` is meaningless to anyone else. If your team wants author namespacing, combine it with the type prefix:

```
john/feature/user-auth    # acceptable if team convention
feature/user-auth          # simpler and usually sufficient
```

### 7. Be consistent above all else

Pick a convention and enforce it. A mediocre convention used consistently beats a perfect one used sometimes.

## Long-lived vs. Short-lived Branches

**Long-lived branches** (`main`, `develop`, `staging`, `production`) should have simple, fixed names. Don't apply the prefix convention to these.

**Short-lived branches** (everything else) use the `type/description` format and get deleted after merging. These are the branches where naming conventions matter most because there will be dozens or hundreds of them over time.

## Practical Shell Tips

Filter branches by type:
```bash
git branch --list "feature/*"
git branch --list "bugfix/*"
```

Delete all merged feature branches:
```bash
git branch --merged main | grep 'feature/' | xargs git branch -d
```

See who has branches and what kind:
```bash
git branch -r | sort
```

Find a branch by keyword:
```bash
git branch --list "*auth*"
```

## Enforcing Conventions

### Git Hook (pre-commit)

Drop this in `.git/hooks/pre-commit` (or use a shared hooks directory):

```bash
#!/bin/bash
branch=$(git symbolic-ref --short HEAD)
pattern="^(feature|bugfix|hotfix|refactor|docs|test|chore|release)\/[a-z0-9][a-z0-9\-]*$"

if ! echo "$branch" | grep -Eq "$pattern"; then
    echo "ERROR: Branch name '$branch' doesn't follow convention."
    echo ""
    echo "Expected: <type>/<description>"
    echo "Types: feature, bugfix, hotfix, refactor, docs, test, chore, release"
    echo "Description: lowercase alphanumeric with hyphens"
    echo ""
    echo "Examples:"
    echo "  feature/user-auth"
    echo "  bugfix/PROJ-123-fix-login"
    echo "  hotfix/crash-on-submit"
    exit 1
fi
```

Make it executable:
```bash
chmod +x .git/hooks/pre-commit
```

### CI Check (GitHub Actions snippet)

```yaml
- name: Check branch name
  run: |
    BRANCH="${GITHUB_HEAD_REF:-${GITHUB_REF#refs/heads/}}"
    if ! echo "$BRANCH" | grep -Eq "^(feature|bugfix|hotfix|refactor|docs|test|chore|release)/[a-z0-9][a-z0-9-]*$"; then
      echo "Branch '$BRANCH' does not match naming convention"
      exit 1
    fi
```

## Quick Reference

When advising on branch names, follow this decision tree:

1. **What kind of work is it?** -> Pick the prefix (`feature/`, `bugfix/`, etc.)
2. **Is there a ticket?** -> Include the ID after the prefix
3. **What's the short summary?** -> 2-4 hyphenated words describing the change
4. **Is it under ~50 chars?** -> If not, trim it down

### Common Mistakes to Flag

- Using `main` or `master` for feature work
- Branch names with spaces (git won't allow it, but people try)
- CamelCase or mixed case
- Generic names: `test`, `fix`, `update`, `wip`, `temp`
- Overly long descriptive names
- Inconsistent prefixes across the same team (mixing `bug/` and `bugfix/` and `fix/`)

## References

- [SO: Common practices for naming git branches](https://stackoverflow.com/questions/273695/what-are-some-examples-of-commonly-used-practices-for-naming-git-branches)
- [Git Style Guide (based on SO answer)](https://github.com/chrisl8888/git-style-guide)
- [Conventional Branch spec](https://conventional-branch.github.io/)
