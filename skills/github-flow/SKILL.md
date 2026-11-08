---
name: github-flow
description: Use when creating a branch, committing, bumping project's version, pushing, or opening a pull request in a repository.
---

# Github Flow

Branches, commits, versions and PR's in any github repositories.

## Types
`feat | fix | docs | style | refactor | test | chore | perf | build | ci`

## 1. Branch
- Always branch from an up-to-date `main`: `git fetch origin main && git switch -c <branch> origin/main`.
- Format: `<type>/<short-description>`, with the description in lowercase kebab-case. Example: `feat/create-users-crud`.
- Already on a branch for this task: reuse it.

## 2. Commit
- Format: `<type>(<scope>): <short description>`, lowercase, imperative, **≤ 50 characters in total**. Example: `feat(auth): add jwt validation to login endpoint`.
- `scope` = the module or domain touched (`calendars`, `employees`, `auth`).
- Check the length before committing; if it exceeds 50, shorten the description, never the type.
- No trailers, no attribution lines.

## 3. Push
- Before pushing, confirm the branch contains the latest main: `git fetch origin main && git merge-base --is-ancestor origin/main HEAD`. If it doesn't, rebase on `origin/main` and ask before any force push.
- `git push -u origin <branch>`.

## 4. Pull request
- Title: `<Short Task Title>: <Short description of changes>`.
- Description, exactly this structure, ending with the tag:
```markdown
## Description of the changes
<What this change does and why it is necessary.>
```
## Output
Report only the steps that were run:
```
Status: DONE | PARTIAL | BLOCKED
Repo: <absolute path>   Task: CU-<id>
Branch: <branch> (from origin/main @ <short sha>)
Commits:
- <short sha> <message>
Version: <old> → <new> (<LEVEL>: <reason>) | unchanged
Push: pushed | failed: <reason> | not requested
MR: !<iid> <url> | not requested
Checks: target main ✓|✗, assignee sebastian298 ✓|✗, label Back ✓|✗, delete source ✓|✗, squash ✓|✗
Blocked by: <what needs the user: missing CU id, version level, force push approval, SSH key>
```

## Never
- Target a branch other than `main`, or branch from anything but `main`.
- Commit directly to `main`, or force push without asking.
- Commit secrets, `.env` files or local config changes that aren't part of the task.
