---
name: interview
description: Interview skill that uses AskUserQuestion to clarify prompts
---

## Summary
Use the AskUserQuestionTool to remove any ambiguity with the prompt.

## Closing Question
After all clarifying questions have been answered, ALWAYS end with one final AskUserQuestion: **"Is there anything else you'd like to add or any extra requirements that came to mind?"** — this gives the user a chance to provide additional context or requirements they thought of while answering. Only proceed to implementation after this final question is answered.

## Special case
- If the prompt mentions anything about a plan document or prd file then read the document or file and interview me in detail using the AskUserQuestionTool about literally anything: technical implementation, UI & UX, concerns, tradeoffs, etc. Really go deep in this case.
- This special planning interview should begin by explicitly telling the user that we are about to go deep on the plan document by using its name if applicable.