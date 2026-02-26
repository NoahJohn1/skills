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

### Step 4 — Summarize changes

Run `git log main..HEAD --oneline` to list commits on this branch. Use the commit messages to write a short (2–5 bullet) summary of changes for the PR body.

### Step 5 — Create the PR

Run:
```
gh pr create \
  --repo <owner>/<repo> \
  --title "<issue title>" \
  --body "Closes #<number>

## Summary
<bullet points from Step 4>"
```

### Step 6 — Output

Print the PR URL so the user can open it directly.

## Notes

- `Closes #<number>` in the body is what GitHub uses to auto-link and auto-close the issue when the PR merges
- If a PR already exists for this branch, `gh pr create` will error — handle by running `gh pr view --repo <owner>/<repo>` and printing the existing PR URL instead
- Do not push the branch — assume the user has already pushed, or let `gh pr create` handle it (it will prompt if needed)
