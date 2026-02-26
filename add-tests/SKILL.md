---
name: add-tests
description: Generate unit tests for code. Use when tests are needed for functions, classes, or modules.
allowed-tools: Read, Grep, Glob, Edit, Write, Bash
---

Generate tests for: $ARGUMENTS

## Approach

1. **Detect framework**: Find existing test patterns in codebase
2. **Identify test cases**: Apply the usefulness filter (see below)
3. **Write tests**: Match project's testing style
4. **Evaluate tests**: Remove any that fail the usefulness criteria
5. **Run tests**: Verify they pass

## Test Usefulness Criteria

Before writing each test, ask: **"What bug would this catch?"**

### A test IS useful when it:
- **Catches real bugs** - tests boundary conditions, null cases, error paths
- **Documents behavior** - shows how the code is supposed to work
- **Enables safe refactoring** - fails when behavior changes unintentionally
- **Covers decision points** - each if/else, validation, branching logic
- **Tests the contract** - inputs → expected outputs

### A test is NOT useful when it:
- **Tests implementation, not behavior** - private methods, internal state, call order
- **Duplicates another test** - same code path exercised twice
- **Tests trivial code** - getters/setters, pass-through methods, obvious defaults
- **Has low ROI** - complex setup for unlikely edge case
- **Tests framework code** - verifying ORM inserts correctly, logger logs
- **Tests configuration wiring** - that config values are read correctly
- **Verifies call sequences** - order of method calls (unless order is a business rule)

## Test Priorities (by value)

1. **Business rules** - validation logic, calculations, state transitions
2. **Error handling** - what happens when things fail
3. **Boundaries** - 0, 1, many, null, empty, max values
4. **Happy path** - the main success scenario
5. **Integration points** - external service calls (mocked)

## What to Skip

- Logging verification
- Call count verification (unless critical)
- Implementation details (GUID format, internal data structures)
- Trivial initializations (empty lists, default values)
- Config reading mechanics

## Rules

- Follow existing test file naming conventions
- One logical assertion per test
- Descriptive test names: `MethodName_Scenario_ExpectedResult`
- Arrange-Act-Assert pattern
- Mock external dependencies, not internal logic

## After Writing Tests

Review each test and ask:
1. What production bug would this catch?
2. Is this testing behavior or implementation?
3. Does another test already cover this path?

If you can't answer #1, or #2 is "implementation", or #3 is "yes" - delete the test.
