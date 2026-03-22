---
name: work-on-issue
description: Fetch a GitHub issue by number, interview to fill gaps, then enter plan mode for approval before implementation.
allowed-tools: Bash(*), Read, Grep, Glob, Edit, Write, Task, AskUserQuestion, EnterPlanMode, Skill
---

# Work On Issue

Fetches a GitHub issue, fills in any ambiguity via interview, creates a branch, and enters plan mode so the user can approve before implementation begins.

## Usage

```
/work-on-issue <issue-number>
```

## Workflow

### Step 1 — Resolve repo

Read the project-level `CLAUDE.md` (in the current working directory) to find the `--repo` value. Look for a line matching `--repo <owner>/<repo>`. If not found, fall back to parsing `git remote get-url origin`.

### Step 2 — Fetch issue

Run:
```
gh issue view <number> --repo <owner>/<repo> --json number,title,body,labels,comments
```

Display a clean summary:
- **Title**: `#<number> <title>`
- **Labels**: comma-separated label names (if any)
- **Body**: full issue body
- **Comments**: each comment's author and body, in order (if any)

### Step 3 — Create branch

Generate a slug from the issue title:
- Lowercase, hyphens only, max 50 chars
- Strip punctuation and special characters
- Branch name format: `<number>-<slug>` e.g. `42-fix-navbar-scroll`

Run:
```
git checkout -b <branch-name>
```

Confirm the branch was created and switched to.

### Step 4 — Interview to fill gaps

Call `Skill("i")` to run the deep-dive interview. The `i` skill will relentlessly ask questions until all ambiguity is eliminated. Focus areas:
- What "done" looks like (acceptance criteria if not specified)
- Any technical approach decisions the issue leaves open
- Edge cases or constraints not mentioned
- UI/UX details if the issue touches the frontend

Do NOT skip straight to planning — the `i` skill must complete first.

### Step 5 — Detect frontend involvement

After the interview, check whether the issue involves any frontend work. An issue is frontend-related if it mentions: screens, UI, components, layout, styling, navigation, forms, modals, lists, buttons, icons, colors, fonts, animations, or any user-facing visual element.

If the issue IS frontend-related, call `Skill("frontend")` before entering plan mode. The `frontend` skill will handle UX analysis and design quality — let it complete fully before proceeding.

### Step 6 — Enter plan mode

After the interview (and frontend skill if applicable), call `EnterPlanMode`. In plan mode:
- Explore the codebase relevant to the issue
- Write a concrete implementation plan to the plan file
- The user approves the plan via `ExitPlanMode`
- After approval, context is cleared and Claude implements the plan

## After Implementation

Once coding is complete and changes are committed, remind the user to run:
```
/submit-issue
```
This will auto-detect the issue number from the branch name and create a linked PR.

## Notes

- Always read project CLAUDE.md first for repo config — never hardcode the repo
- The branch is created from whatever branch is currently checked out
- Do not start writing code before plan mode approval
- Keep the interview focused — 2–4 questions max unless the issue is very vague
