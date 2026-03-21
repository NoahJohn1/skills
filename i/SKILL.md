---
name: i
description: Deep-dive interview that relentlessly asks questions until every detail of the task is fully understood. Will not stop asking until all ambiguity is eliminated. Use this skill PROACTIVELY whenever the user's prompt is ambiguous, underspecified, or could lead to wasted work — including vague feature requests ("add logging", "refactor auth"), bug reports without repro steps, multi-step tasks touching multiple files/systems, architecture or design decisions, or any task where jumping straight to code risks building the wrong thing. Even if the user doesn't say "interview me" — if their request leaves important questions unanswered, invoke this skill before writing code.
---

## Summary
Use AskUserQuestion to remove ALL ambiguity from the user's request before implementation begins. The goal is to fully understand the task so implementation can succeed on the first attempt.

## Step 0: Read Before You Ask
Before asking a single question, scan the codebase for context relevant to the user's request. Use Glob and Grep to find the files, classes, and patterns involved. This lets you ask informed, specific questions ("I see OrderService already has a RetryPolicy — should the new retry logic follow that pattern or replace it?") instead of generic ones ("How should retries work?"). Spending 30 seconds reading code saves multiple rounds of back-and-forth.

## Smart Batching
Use AskUserQuestion to batch **independent** questions together (up to 4 per round). Independent means: the answer to one question won't change what you'd ask in another. If a question's answer could redirect the entire task, ask it alone first, then batch the follow-ups.

**Good batch** (independent): "What's the expected input format?" + "Who will consume this output?" + "Any performance constraints?"

**Bad batch** (dependent): "Should we use a queue or direct API call?" + "What retry policy for the queue?" — Q2 is only relevant if the answer to Q1 is "queue."

## Auto-Calibrate Depth
Not every task needs 10 rounds of questions. Judge the complexity:

- **Small/clear task** (rename a variable, fix a typo, add a simple field): 1 round or skip interviewing entirely if the task is obvious
- **Medium task** (new endpoint, bug fix with unclear scope, refactor): 2-3 rounds covering what/why/where/how
- **Large/complex task** (new feature, cross-system change, architecture decision): full deep dive covering all dimensions below

If you're unsure, err toward asking more rather than less — the cost of building the wrong thing is always higher than the cost of a few extra questions.

## Dimensions to Cover (for medium-to-large tasks)
- **What** exactly needs to happen (functional requirements)
- **Why** this is needed (context, motivation, business reason)
- **Where** in the codebase this lives (files, projects, layers)
- **How** it should behave in edge cases and error scenarios
- **Who** is affected (users, systems, downstream consumers)
- **Acceptance criteria** — how do we know it's done?
- **Out of scope** — what should this NOT do?

You don't need to mechanically hit every dimension — use judgment. If the user's prompt already answers some of these, skip them and focus on the gaps.

## Question Strategy
1. Start broad: understand the goal and context
2. Narrow down: get into specifics, implementation details, constraints
3. Probe edges: error handling, edge cases, "what if X happens?"
4. Validate assumptions: repeat back your understanding and ask if it's correct
5. Final sweep: ask about anything you haven't covered

- If an answer raises new questions or implies complexity, follow that thread immediately.
- If an answer is vague, push back and ask for specifics. Do not accept hand-wavy answers.
- Track what you've learned and what gaps remain.

## Closing Sequence
After you believe all questions have been answered:
1. Summarize your complete understanding of the task in a concise bullet list.
2. Ask: **"Here's my understanding of the full task. Is anything wrong, missing, or incomplete?"**
3. If the user corrects or adds anything, follow up on those additions with more questions if needed.
4. End with one final AskUserQuestion: **"Is there anything else you'd like to add or any extra requirements that came to mind?"**

## Plan Mode Transition
After the interview is complete, assess the task size. If the task is medium-to-large (touches multiple files, involves new patterns, or has significant complexity), suggest entering Plan Mode so that a structured implementation plan can be created and unneeded interview context can be cleared. Frame it as: "This is a sizeable task — want me to enter Plan Mode to lay out an implementation plan before we start coding?"

For small tasks, skip this and proceed directly to implementation.

## Special Cases
- **Plan documents / PRD files**: If the prompt mentions a plan document or PRD, read the document first, then interview in detail about technical implementation, UI & UX, concerns, tradeoffs, etc. Begin by telling the user you're about to go deep on the plan document by name.
- **Bug reports**: Focus questions on reproduction steps, expected vs actual behavior, environment details, and whether a fix already exists elsewhere in the codebase.
- **Architecture decisions**: Focus on constraints, tradeoffs, existing patterns in the codebase, and reversibility of the decision.
