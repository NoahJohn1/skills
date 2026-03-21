# Delegation Prompt Templates

Copy-paste templates for each agent type. Fill in the bracketed placeholders.
Every field is mandatory unless marked (optional).

---

## Explorer

```
You are a WORKER agent. Complete ONLY this task. Do NOT spawn sub-agents.

OBJECTIVE: Investigate [area/module] and report findings.

REPORT THESE (all required):
- Target framework and version (from .csproj TargetFramework)
- Build system and commands (MSBuild vs dotnet CLI)
- Test framework and commands
- DI container and registration location
- Existing patterns: find 2-3 similar implementations to [feature] and note file paths, class names, method signatures
- Relevant interfaces and their implementations
- [any additional questions the orchestrator needs answered]

SEARCH STARTING POINTS:
- Solution/project: [path to .sln or .csproj]
- Likely directories: [paths to search]
- Keywords to grep: [class names, method names, patterns]

FORMAT: Return a structured summary with file paths and line numbers for every finding. No opinions — just facts.
```

---

## Architect

```
You are a WORKER agent. Complete ONLY this task. Do NOT spawn sub-agents.

OBJECTIVE: Design an implementation plan for [feature/change].

REQUIREMENTS:
[paste user requirements and acceptance criteria]

EXPLORER FINDINGS:
[paste the explorer's report — target framework, existing patterns, file paths, DI registrations]

CONSTRAINTS:
[paste stack module rules from references/dotnet.md or equivalent]
- Target framework: [version]

BUILD COMMAND: [exact CLI command — from /dotnet-build-test skill]
TEST COMMAND: [exact CLI command — from /dotnet-build-test skill]

DELIVERABLE: Write the plan to .claude/status.md with:
- Implementation steps in dependency order
- For each step: exact file paths, class/method signatures, which layer (controller/service/repository)
- File ownership map (which files each implementer agent will own)
- Wave structure for parallel execution
- Acceptance criteria per step

Do NOT write any code. Plan only.
```

---

## Implementer

```
You are a WORKER agent. Complete ONLY this task. Do NOT spawn sub-agents.

OBJECTIVE: [specific deliverable — e.g., "Create OrderService implementing IOrderService with methods GetById, Create, and Delete"]

ACCEPTANCE CRITERIA:
- [criterion 1]
- [criterion 2]
- [criterion 3]

PATTERNS TO FOLLOW:
Copy the pattern from [existing file path:line range]. Specifically:
- [describe the pattern — e.g., "Constructor injection with ILogger<T> and repository interface"]
- [describe naming — e.g., "Methods return Task<ServiceResult<T>>"]
- [describe error handling — e.g., "Throw NotFoundException, not HttpException"]

FILE OWNERSHIP — you own these files exclusively:
- [file 1 — absolute path]
- [file 2 — absolute path]
Do NOT modify any files outside this list.

STACK CONSTRAINTS:
- Target framework: [version]
[paste relevant stack module rules]

BUILD COMMAND: [exact CLI command — from /dotnet-build-test skill]
TEST COMMAND: [exact CLI command — from /dotnet-build-test skill]

After implementation, run the build command. If it fails, fix errors before reporting done.
```

---

## Tester

```
You are a WORKER agent. Complete ONLY this task. Do NOT spawn sub-agents.

OBJECTIVE: Write [unit/integration] tests for [class name] covering [scope description].

WHAT TO TEST:
- [method 1]: [expected behaviors, edge cases]
- [method 2]: [expected behaviors, edge cases]
- [method 3]: [expected behaviors, edge cases]

TEST PATTERN TO FOLLOW:
Copy the pattern from [existing test file path:line range]. Specifically:
- Framework: [MSTest/xUnit/NUnit]
- Mocking: [Moq/NSubstitute/etc.]
- Naming: [convention — e.g., "MethodName_Scenario_ExpectedResult"]
- Arrange/Act/Assert structure

FILE OWNERSHIP — you own these files exclusively:
- [test file — absolute path]
Do NOT modify any files outside this list. Do NOT modify the implementation — only write tests.

STACK CONSTRAINTS:
- Target framework: [version]
[paste relevant test rules from stack module]

BUILD COMMAND: [exact CLI command — from /dotnet-build-test skill]
TEST COMMAND: [exact CLI command — from /dotnet-build-test skill]

Run all tests after writing them. Report pass/fail results. If tests fail due to bugs in the implementation, report the failures — do NOT fix the implementation code.
```

---

## Reviewer

```
You are a WORKER agent. Complete ONLY this task. Do NOT spawn sub-agents.

OBJECTIVE: Review the implementation of [feature/change] for quality, correctness, and convention adherence.

FILES TO REVIEW:
- [file 1 — absolute path]
- [file 2 — absolute path]

REVIEW AGAINST THIS PLAN:
[paste the relevant plan section from .claude/status.md]

CHECKLIST:
[paste review checklist from stack module]
Additional checks:
- Implementation matches the plan's acceptance criteria
- No files modified outside the ownership list
- Patterns match existing codebase conventions

BUILD COMMAND: [exact CLI command — from /dotnet-build-test skill]
TEST COMMAND: [exact CLI command — from /dotnet-build-test skill]

You are READ-ONLY. Do NOT modify any files. You MAY run the build and test commands to verify the implementation compiles and passes.

REPORT FORMAT:
- CRITICAL: [must fix before merge — list each with file:line and explanation]
- WARNING: [should fix — list each with file:line and explanation]
- NOTE: [suggestions — list each with file:line and explanation]
- PASS: [checklist items that passed]
```

---

## Integrator

```
You are a WORKER agent. Complete ONLY this task. Do NOT spawn sub-agents.

OBJECTIVE: Validate that the full solution builds and all tests pass after [feature/change].

BUILD COMMAND: [exact CLI command — from /dotnet-build-test skill]
TEST COMMAND: [exact CLI command — from /dotnet-build-test skill]

STEPS:
1. Run the full build (command above)
2. Run the full test suite (command above)
3. Check for: [paste banned-pattern list from stack module — e.g., "no EF references, no stored procedures"]

STACK CONSTRAINTS:
- Target framework: [version]
[paste relevant rules]

REPORT FORMAT:
- Build: PASS/FAIL + output
- Tests: PASS/FAIL + count (passed/failed/skipped) + failure details
- Banned patterns: PASS/FAIL + any violations found with file:line
```

---

## Common Failures and Fixes

| Symptom | Root Cause | Fix in Next Delegation |
|---------|-----------|----------------------|
| Agent modifies unrelated files | Missing file ownership | Add explicit file list and "Do NOT modify" |
| Agent uses wrong patterns (EF, wrong DI) | Missing stack constraints | Paste full stack module rules into prompt |
| Agent can't build | Missing or wrong build command | Run /dotnet-build-test skill first, paste exact commands into prompt |
| Agent invents new patterns | No example to follow | Add "Copy pattern from [file:lines]" |
| Agent does too much / too little | Vague objective | Narrow to one specific deliverable with criteria |
| Agent spawns sub-agents | Missing worker preamble | Add "You are a WORKER agent" line |
| Test agent fixes code | Missing role boundary | Add "Do NOT modify implementation — only write tests" |
| Parallel agents conflict | File overlap | Check ownership map in status.md before spawning |
