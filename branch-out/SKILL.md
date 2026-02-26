---
name: branch-out
description: Create git worktree for isolated task work. Use when starting work on a task that needs separate workspace.
allowed-tools: Bash(*), Read, Grep, Glob, Edit, Write, Task, TodoWrite, AskUserQuestion
---

# Branch Out - Worktree Creation Skill

Creates an isolated git worktree for task-based development, auto-generating branch name from user's prompt.

## Usage

```
/branch-out <task description>
```

The user's prompt becomes the task context. Branch name is auto-generated as a slug.

## What This Skill Does

1. **Parses Prompt**: Extracts task intent from user message
2. **Generates Slug**: Creates branch name like `add-dark-mode` or `fix-login-bug`
3. **Creates Worktree**: Sets up isolated workspace in `.claude/temp-workspaces/<slug>`
4. **Creates Branch**: Named after slug from current branch
5. **Switches Context**: Moves into worktree directory
6. **Starts Work**: Immediately begins implementation

## Workflow

When invoked:

1. Parse user prompt to understand task: `$ARGUMENTS`
2. Generate slug from task (lowercase, hyphens, max 50 chars)
3. Get current branch name (source branch)
4. Create worktree: `git worktree add .claude/temp-workspaces/<slug> -b <slug>`
5. Change directory: `cd .claude/temp-workspaces/<slug>`
6. Display task context and source branch
7. **Immediately begin implementation** of the task

## Branch Naming

Auto-generate slug from prompt:
- `"Add dark mode toggle"` → `add-dark-mode-toggle`
- `"Fix login bug with email validation"` → `fix-login-bug`
- `"Refactor user service"` → `refactor-user-service`

Keep it concise, lowercase, hyphenated, descriptive.

## Important Notes

- **Isolation**: All work happens in worktree, main directory untouched
- **Source Branch**: Worktree branches from currently active branch
- **Cleanup**: Use `/branch-in` to merge and cleanup
- **Multiple Tasks**: Can have multiple worktrees simultaneously
- **Auto-Start**: Begins working immediately after setup

## Example

```
User: /branch-out Add dark mode toggle to settings page
→ Task: Add dark mode toggle to settings page
→ Branch: add-dark-mode-toggle
→ Creating worktree: .claude/temp-workspaces/add-dark-mode-toggle
→ Branching from: main
→ [Switches to worktree and begins implementation]
```

## Protocol

Follow this exact sequence:

1. Parse `$ARGUMENTS` to understand task
2. Generate branch slug (lowercase-with-hyphens)
3. Capture current branch as source branch
4. Create worktree: `.claude/temp-workspaces/<slug>`
5. Create and checkout new branch `<slug>`
6. Change directory: `cd .claude/temp-workspaces/<slug>`
7. Display: task description, branch name, source branch
8. **Begin implementation immediately** - this is critical
9. Use TodoWrite if task is complex enough to warrant it

## Post-Setup

After creating worktree, treat the user's original prompt as the implementation request and begin working on it immediately in the worktree context.
