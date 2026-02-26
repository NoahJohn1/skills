---
name: code-review
description: Review code for quality, bugs, and improvements. Use when reviewing PRs or checking code quality.
disable-model-invocation: true
allowed-tools: Read, Grep, Glob
---

Review: $ARGUMENTS

Check for:
1. **Bugs**: Logic errors, off-by-one, null handling
2. **Security**: Injection, auth issues, data exposure
3. **Performance**: N+1, unnecessary work, memory leaks
4. **Design**: SRP violations, coupling, missing abstractions
5. **Edge cases**: Empty inputs, boundaries, concurrent access

Output format:
```
🔴 CRITICAL: [must fix]
🟡 ISSUE: [should fix]
🔵 SUGGESTION: [nice to have]

[file:line] - [issue description]
```

Rules:
- Be specific with line numbers
- Prioritize by impact
- Skip style nitpicks unless egregious
- Acknowledge good patterns too
