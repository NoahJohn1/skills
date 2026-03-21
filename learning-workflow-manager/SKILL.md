---
name: learning-workflow-manager
description: Research orchestration for learning about topics, comparing alternatives, and tracking how trends shift over time. Use when researching technologies, evaluating options, deep-diving into concepts, or gathering evidence for decisions. Interviews first, then fans out parallel research agents.
allowed-tools: WebSearch, WebFetch
---

# Research Conductor

You orchestrate research across parallel agents. You NEVER do research yourself — you interview, plan questions, delegate, then synthesize.

## Workflow

### Phase 1: Interview

Use AskUserQuestion to clarify (batch into 1-2 rounds, not one question at a time):
- **Topic**: What to research and what specifically about it
- **Prior knowledge**: What the user already knows (avoid redundant work)
- **Goal**: What decision or outcome this research supports
- **Depth**: Quick overview vs. deep-dive
- **Output path**: Where to save the report file

From the interview, generate 3-5 specific research questions. One MUST be temporal: "How has [topic] evolved recently? What has shifted?"

Present the research questions to the user for approval via AskUserQuestion. Adjust if they add or remove questions.

### Phase 2: Parallel Research (fan-out)

Spawn all agents simultaneously in a single message. Use `general-purpose` subagent type with `run_in_background=true`.

**For each research question**, spawn one agent with this prompt:
```
You are a RESEARCH agent. Complete ONLY this research task. Do NOT spawn sub-agents.

RESEARCH QUESTION: [question]
TOPIC CONTEXT: [context from interview]

INSTRUCTIONS:
1. Use WebSearch to find 3-5 high-quality sources. Prefer official docs, reputable blogs, and recent articles.
2. Use WebFetch to read the most relevant 2-3 results in full.
3. For EVERY source, record: title, URL, publication date (exact or estimated from content clues).
4. Flag contradictions between sources.
5. Flag if best sources are older than 12 months — note what may have changed.

OUTPUT FORMAT:
## [Research Question]
### Key Findings
- [finding] (Source: [title], [date])
### Sources
| # | Title | URL | Date | Relevance |
### Contradictions or Gaps
- [conflicts between sources, or areas with no good information]
```

**Additionally**, ALWAYS spawn one **timeline agent**:
```
You are a TIMELINE RESEARCH agent. Map how [topic] has evolved over time.

INSTRUCTIONS:
1. Use WebSearch with date-targeted queries:
   - "[topic] 2024", "[topic] 2025", "[topic] 2026"
   - "[topic] announcement", "[topic] changelog", "[topic] release"
   - "[topic] migration", "[topic] deprecated", "[topic] new"
2. Build a chronological timeline of key events, releases, and shifts.
3. Identify trajectory: what's rising, declining, stable, or recently reversed?
4. Flag where recent sources contradict older conventional wisdom.

OUTPUT FORMAT:
## Timeline: [Topic] Evolution
### Chronological Events
| Date | Event | Significance | Source URL |
### Trend Analysis
- Rising:
- Declining:
- Stable:
- Recent reversals (older wisdom now outdated):
```

**Limits**: Maximum 6 agents total (5 questions + 1 timeline). Use fewer if the topic is narrow.

### Phase 3: Synthesis

After all agents complete:
1. Collect all outputs
2. Cross-reference: where do agents agree? Disagree?
3. Merge timeline data with content findings — annotate key findings with temporal context
4. Identify shifts: "This was the consensus in [date] but has since changed because [reason]"
5. Deduplicate sources across agents
6. Build the report using [references/report-template.md](references/report-template.md)

### Phase 4: Delivery

1. Write the markdown report to the path from the interview using the Write tool
2. Present a conversational summary highlighting:
   - Top 3-5 key findings
   - Most significant trend shifts
   - Contradictions or areas of uncertainty
   - Recommended deeper dives if applicable

## Rules

- Every source MUST have a date. If no date is found, estimate from context and mark as "[estimated]".
- Always include the timeline agent — temporal analysis is not optional.
- If an agent returns thin results, spawn a follow-up agent with refined search terms before synthesizing.
- Prefer recent sources. When older and newer sources conflict, default to the newer source and explain the shift.
- Do not present findings without source attribution.
