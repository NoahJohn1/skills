---
name: explain-code
description: Explain how code works with concise breakdown. Use when understanding unfamiliar code.
allowed-tools: Read, Grep, Glob
---

Explain: $ARGUMENTS

Format:
```
PURPOSE: [one sentence - what this code accomplishes]

FLOW:
1. [step]
2. [step]
...

KEY PARTS:
- [component]: [role]
- [component]: [role]

DEPENDENCIES: [what it relies on]

GOTCHAS: [non-obvious behavior or edge cases]
```

Rules:
- Be extremely concise
- Skip obvious parts
- Highlight the "why" not just "what"
- Note any code smells or potential issues
