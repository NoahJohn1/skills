---
name: create-issue
description: Interactively interview the user to define requirements for a GitHub issue, then create it.
allowed-tools: Bash(*), Read, Grep, Glob, AskUserQuestion
---

# Create Issue

Interviews the user to flesh out a well-defined GitHub issue, then creates it via the gh CLI.

## Usage

```
/create-issue <rough idea or title>
```

The user's prompt is a starting point — not necessarily the final title or full spec.

## Workflow

### Step 1 — Understand the rough idea

Parse `$ARGUMENTS` as the user's initial idea. If empty, ask them for one first.

### Step 2 — Interview to define requirements

Use `AskUserQuestion` to help the user think through the issue. Tailor questions based on the type of issue (bug vs. feature vs. chore). Ask 2–4 focused questions from the relevant set below — do not ask all of them, only the ones that are genuinely unclear given the prompt.

**For features / enhancements:**
- What is the user-facing goal? What problem does this solve?
- What does "done" look like — what can the user do that they couldn't before?
- Are there edge cases, constraints, or out-of-scope items to note?
- Any UI/UX considerations or design requirements?

**For bugs:**
- What is the current (broken) behavior?
- What is the expected behavior?
- Are there steps to reproduce?
- What is the impact / severity?

**For chores / refactors:**
- What is the motivation (performance, maintainability, tech debt)?
- What is the scope — which files or systems are affected?
- Any risks or things to be careful about?

End with the closing question: **"Is there anything else you'd like to add or any extra requirements that came to mind?"**

### Step 3 — Synthesize into issue fields

From the interview answers, compose:

- **Title**: concise, imperative (e.g. "Add dark mode toggle to settings page")
- **Body**: structured markdown with sections:
  ```
  ## Description
  <what and why>

  ## Acceptance Criteria
  - [ ] <criterion 1>
  - [ ] <criterion 2>

  ## Notes
  <edge cases, constraints, out-of-scope, etc. — omit section if nothing to add>
  ```
- **Labels**: suggest appropriate labels based on issue type (bug, enhancement, chore) — only include if they exist in the repo

Show the composed title and body to the user before creating, so they can see what will be submitted.

### Step 4 — Confirm and create

Ask the user to confirm with a final `AskUserQuestion`:
- **"Create this issue as shown?"** — Yes / Edit title / Edit body / Cancel

If confirmed, run:
```
gh issue create \
  --repo <owner>/<repo> \
  --title "<title>" \
  --body "<body>"
```

Resolve `--repo` from project `CLAUDE.md` (line matching `--repo <owner>/<repo>`), falling back to `git remote get-url origin`.

### Step 5 — Output

Print the created issue URL. Mention that they can start working on it with `/work-on-issue <number>`.

## Notes

- Keep the interview conversational — don't make it feel like a form
- The goal is to help the user surface requirements they haven't thought about yet, not just transcribe what they said
- If the prompt is already detailed enough, skip questions that are already answered
