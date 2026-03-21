# Orchestration Patterns

Patterns for parallel and sequential task coordination.

## Task Graph (DAG)

The foundation pattern. All other patterns are special cases of a task graph.

Define tasks with dependency edges, then walk the graph:

1. Find all unblocked pending tasks (no incomplete dependencies)
2. Spawn agents for unblocked tasks (up to 10 parallel)
3. When one completes, check for newly unblocked tasks
4. Spawn agents for those
5. Repeat until all tasks complete

### Task Descriptions

Write each task description as a self-contained agent prompt. Include:
- What to do (specific action and acceptance criteria)
- Where (absolute file paths)
- Context (what to read first, relevant patterns to follow)
- Constraints (stack module rules, build/test commands)
- Deliverables (what files to create or modify)
- File ownership ("You own these files exclusively: [list]. Do NOT modify any files outside this list.")

## File Partitioning

NEVER assign the same file to multiple parallel agents. Violations cause merge conflicts and lost work.

### Partitioning Strategies
- **By layer**: All repository files to one agent, all service files to another
- **By feature/entity**: All Order files (repo + service + controller) to one agent, all Customer files to another (only when no cross-entity dependencies)
- **By module**: Entire module directories to one agent

### Ownership in Prompts

Every delegation prompt MUST include:
```
You own these files exclusively: [list]. Do NOT modify any files outside this list.
```

### Before Each Parallel Spawn

Check `.claude/status.md` file ownership map. If ANY file overlaps between tasks, they cannot run in parallel. Resolve by making one sequential.

## Wave-Based Implementation

For layered architectures (Repository → Service → Controller), implementation proceeds in dependency waves. Within each wave, all file-disjoint tasks run in parallel.

### Example: Feature touching Orders and Customers

```
Wave 1 (parallel): OrderRepository + CustomerRepository implementers (2 agents)
Wave 2 (parallel): OrderService + CustomerService implementers + Wave 1 testers (4 agents)
Wave 3 (parallel): OrderController + CustomerController implementers + Wave 2 testers (4 agents)
Wave 4 (parallel): Wave 3 testers + DI registration implementer (3 agents)
Wave 5 (sequential): reviewer, then integration build + full test suite
```

### What Parallelizes Safely
- Repository implementations for different entities
- Service implementations for different entities (after their repos exist)
- Controller implementations for different entities (after their services exist)
- Tests for completed implementations alongside next-wave implementers
- Multiple explorer investigations of different modules

### What Must Stay Sequential
- A service that depends on a repository not yet built
- A controller that depends on a service not yet built
- DI registration (always after all implementations)
- Review and integration (always last, in that order)
- Any two tasks touching the same file

## Spawning Language

Be EXPLICIT when delegating parallel work.

**DO**: "Execute these [N] tasks in parallel using [N] separate subagents simultaneously."
**DO NOT**: "You can run these in parallel" — too passive, may not trigger parallel execution.

Maximum: 10 subagents in parallel. Use this capacity proactively.
