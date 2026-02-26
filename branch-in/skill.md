---
name: branch-in
description: Merge worktree changes back to source branch for review. Use when task work is complete and ready to integrate.
allowed-tools: Bash(*)
disable-model-invocation: true
---

# Branch In - Worktree Merge Skill

Merges worktree changes back to the source branch, staging them for user review before commit.

## Usage

```
/branch-in
```

No arguments required. Changes are staged on source branch for user to review and commit.

## What This Skill Does

1. **Validates Location**: Confirms we're in a worktree
2. **Runs Tests**: Executes test suite (must pass)
3. **Commits Changes**: Commits any uncommitted work in worktree
4. **Identifies Source**: Determines original branch from worktree metadata
5. **Switches Branch**: Returns to source branch
6. **Squash Merge**: Merges worktree branch with squash (stages but does NOT commit)
7. **Cleanup**: Removes worktree directory and branch

## Workflow

When invoked:

1. Verify we're in a worktree (path contains `.claude/temp-workspaces/`)
2. Extract branch slug from current directory
3. Run tests if applicable (`dotnet test` for .NET, `npm test` for Node)
4. If tests fail, abort and report errors
5. If uncommitted changes exist, stage and commit: `git add -A && git commit -m "WIP: <slug>"`
6. Identify source branch from worktree metadata: `git worktree list --porcelain`
7. Switch to source branch: `git checkout <source-branch>`
8. Squash merge: `git merge --squash <slug>` (stages changes, does NOT commit)
9. Remove worktree: `git worktree remove .claude/temp-workspaces/<slug>`
10. Delete branch: `git branch -D <slug>` (use -D because squash merge breaks ancestry)
11. Show staged changes summary and remind user to review and commit

## Pre-Merge Validation

Before merging:
- Runs project test suite
- Aborts if tests fail
- Reports specific test failures

## Example

```
User: /branch-in
→ Located worktree: add-dark-mode-toggle
→ Running tests...
→ All tests passed ✓
→ Switching to source branch: main
→ Squash merging add-dark-mode-toggle
→ Removing worktree
→ Deleting branch
→ ✓ Changes staged on main - review with `git diff --staged` and commit when ready
```

## Important Notes

- **Must Be In Worktree**: Only works from within `.claude/temp-workspaces/` directory
- **Tests Must Pass**: Won't merge if tests fail
- **Squash Merge**: All worktree commits squashed into staged changes
- **No Auto-Commit**: Changes are staged for user review, NOT committed
- **Use -D for Branch Delete**: Squash merges break Git's ancestry tracking, so `-D` (force) is required instead of `-d`

## Protocol

Follow this exact sequence:

1. Verify current directory is in `.claude/temp-workspaces/`
2. Extract branch slug from path or current branch
3. Detect project type and run appropriate tests
4. If tests fail, abort with error details
5. If uncommitted changes exist, commit them with message "WIP: <slug>"
6. Parse worktree metadata to find source branch
7. Switch to source branch
8. Squash merge worktree branch (do NOT commit - leave staged)
9. Remove worktree directory
10. Delete worktree branch with `-D` flag (required for squash-merged branches)
11. Show `git diff --staged --stat` summary
12. Remind user to review and commit

## Error Handling

- Not in worktree → Error: "Must run from worktree in .claude/temp-workspaces/"
- Tests fail → Error with test output, don't merge
- Merge conflicts → Report to user for manual resolution
- Source branch detection fails → Ask user for source branch

## Safety

This command is marked `disable-model-invocation: true` because it:
- Has side effects (merges, deletes branches/worktrees)
- Should be user-initiated
- Requires explicit confirmation via invocation
- Modifies git state
- Irreversible cleanup operations
