---
name: frontend
description: "Create and improve frontend interfaces with expert UX analysis. Automatically spawns the ux-designer agent to evaluate visual design, style, layout, accessibility, and user experience — then implements improvements directly. Use this skill PROACTIVELY whenever the user mentions frontend, UI, page, view, layout, form, modal, navbar, sidebar, table styling, responsive design, CSS, Bootstrap components, Razor views, or any visual/interface work. Even if the user doesn't say 'review the UX' — if they're building or modifying anything visual, invoke this skill."
---

# Frontend Design & UX Skill

## Purpose

You are building or improving a frontend interface. This skill ensures every piece of UI work gets expert UX review and that improvements are implemented automatically — not just recommended.

## Tech Stack Context

The user works primarily with:
- **Razor views** (.cshtml) — remember `@` symbols need escaping (`@@`) in inline JS/Alpine/CDN URLs
- **HTML / CSS**
- **JavaScript / jQuery**
- **Bootstrap** (utility classes, grid, components) — but some apps use jQuery UI instead. Check the existing codebase before assuming Bootstrap is available.

Always check what CSS/JS framework the app actually uses before building. If the app uses jQuery UI dialogs, don't introduce Bootstrap modals. Match the existing patterns.

## Workflow

### Step 1: Understand the Request

Read the relevant view/page files. If the user is asking you to build something new, understand what they need. If they're asking to improve something existing, read the current code first.

Look at the existing views to understand the app's actual styling patterns — what CSS classes are in use, what JS libraries are loaded, what the existing UI conventions are.

### Step 2: Get UX Analysis

**Primary approach — spawn the ux-designer agent:**

Use the Agent tool with `subagent_type: "ux-designer"` to get expert analysis. Pass the agent:

- The current HTML/Razor markup (or a description of what you're building)
- The specific context (what the page does, who uses it, what the user asked for)
- Ask for concrete, actionable feedback on the UX checklist below

**Fallback — inline UX analysis:**

If the Agent tool or ux-designer subagent is not available (e.g., running inside a subagent yourself), perform the UX analysis inline using the same checklist. The analysis should still happen — it just happens in your own reasoning rather than a separate agent.

### UX Checklist

Whether the analysis comes from the ux-designer agent or from your own inline review, evaluate against these dimensions:

- **Visual hierarchy** — is the most important content prominent?
- **Layout & spacing** — consistent padding, alignment, whitespace
- **Typography** — readable sizes, clear headings, font consistency
- **Color & contrast** — accessible contrast ratios, purposeful color use
- **Interactive elements** — buttons, forms, hover states, focus indicators
- **Responsiveness** — does the layout work at different breakpoints?
- **Accessibility** — ARIA labels, keyboard navigation, screen reader compatibility
- **User flow** — is the interface intuitive? Can users accomplish their task efficiently?
- **Empty & loading states** — what does the user see before data loads or when there are no results?
- **Keyboard support** — can power users navigate without a mouse?

### Step 3: Implement the Improvements

Take the UX feedback and implement the changes directly in the code. Do not just list recommendations — apply them.

When implementing:
- Match the existing framework and styling conventions of the app
- Use existing CSS classes and UI patterns from the codebase where possible
- Keep new CSS scoped — use a prefix (e.g., `od-`, `cs-`) to avoid conflicts with existing styles
- Ensure Razor `@` symbols are properly escaped in JS contexts (`@@`)
- Maintain existing functionality while improving the visual/UX layer
- Preserve all existing JS hooks, form fields, onclick handlers, and Razor logic

### Step 4: Summarize What Changed

After implementing, give the user a concise summary:
- What UX issues were identified
- What changes were made
- Any tradeoffs or decisions worth noting

## Brand Guidelines

Only apply Better World Books brand styling if the user explicitly requests it (e.g., "use brand colors," "apply BWB styling"). The user works across multiple apps with different existing styles, so do not assume BWB branding by default. If brand styling is requested, invoke the `brand-guidelines` skill.

## What This Skill Does NOT Do

- It does not replace the user's design judgment — if the user has a specific vision, implement that vision and then offer UX polish suggestions
- It does not force a particular CSS framework — it works with whatever the page already uses
- It does not make functional changes — UX improvements are visual/layout only unless the user asks for behavioral changes
