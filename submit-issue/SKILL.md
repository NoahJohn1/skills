---
name: submit-issue
description: Create a PR for the current branch, auto-linking it to the GitHub issue derived from the branch name.
allowed-tools: Bash(*), Read, Grep, Glob
---

# Submit Issue

Creates a GitHub PR from the current branch, automatically linking it to the issue number embedded in the branch name.

## Usage

```
/submit-issue
```

No arguments needed — the issue number is derived from the branch name.

## Workflow

### Step 1 — Resolve repo

Read the project-level `CLAUDE.md` to find the `--repo` value (line matching `--repo <owner>/<repo>`). Fall back to parsing `git remote get-url origin` if not found.

### Step 2 — Get current branch & extract issue number

Run `git branch --show-current` to get the branch name.

Branch name format is `<number>-<slug>` (e.g. `42-fix-navbar-scroll`). Extract the leading number as the issue number.

If the branch name does not start with a number, warn the user and stop — they may be on the wrong branch.

### Step 3 — Fetch issue title

Run:
```
gh issue view <number> --repo <owner>/<repo> --json title --jq '.title'
```

Use this as the PR title.

### Step 4 — Commit any uncommitted changes

Run `git status --short` to check for uncommitted changes. If any exist:

1. Run `git diff --stat` to see what changed
2. Stage all modified/deleted tracked files: `git add -u`
3. Derive a commit message from the issue title and changed files (e.g. `fix: <short description>`)
4. Commit: `git commit -m "<message>"`

If there is nothing to commit, skip this step.

### Step 5 — Summarize changes

Run `git log main..HEAD --oneline` to list commits on this branch. Use the commit messages to write a short (2–5 bullet) summary of changes for the PR body.

### Step 6 — Create the PR

Run:
```
gh pr create \
  --repo <owner>/<repo> \
  --title "<issue title>" \
  --body "Closes #<number>

## Summary
<bullet points from Step 4>"
```

### Step 7 — Merge the PR

Run:
```
gh pr merge <pr-number> --repo <owner>/<repo> --squash --delete-branch
```

`--delete-branch` removes the remote branch. Wait for the command to succeed before continuing.

### Step 8 — Delete local branch & switch to main

```
git checkout main
git pull
git branch -d <branch-name>
```

### Step 9 — Output

Print the PR URL and confirm: merged, remote branch deleted, local branch deleted.

## Notes

- `Closes #<number>` in the body is what GitHub uses to auto-link and auto-close the issue when the PR merges
- If a PR already exists for this branch, `gh pr create` will error — handle by running `gh pr view --repo <owner>/<repo>` to get the PR number, then proceed to Step 7 with that PR number
- Do not push the branch — assume the user has already pushed, or let `gh pr create` handle it (it will prompt if needed)
