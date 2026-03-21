---
name: doc-it
description: Outputs recent conversation content to a nicely formatted HTML dashboard or AI-optimized Markdown file for viewing documentation. Use when the user wants to save, export, document, or archive conversation content — analysis, comparisons, QA scenarios, requirements, etc. Saves to <repo>/.claude/Docs/ organized by project. Also use when user says "document this", "save this as HTML", "export to markdown", or similar.
---

# Doc It

Convert conversation content into clean, readable documentation files. HTML output renders as a BWB-branded dashboard with sidebar navigation. Markdown output is structured for Claude to consume in future conversations.

## Process

1. **Identify the content to export**:
   - Look at the most recent substantial output in the conversation (comparison docs, requirements, QA scenarios, analysis, summaries, etc.)
   - If ambiguous, use AskUserQuestion to ask which content to export
   - If the user specifies content in their prompt, use that

2. **Ask the user for output format**:
   - Use AskUserQuestion to ask: **HTML**, **Markdown**, or **Both**
   - HTML: BWB-branded dashboard, opens in Chrome, great for human reading and sharing
   - Markdown: AI-optimized, structured for Claude context in future conversations
   - Both: generates paired files in a shared subfolder

3. **Determine the project name**:
   - Derive from the current working directory (use the deepest meaningful folder name, e.g. `IndabaAPI`, `CareDaba`, `BuyBack`)
   - If at a generic path like `source/repos`, use AskUserQuestion to ask for the project name

4. **Determine output path**:
   - Base: `<repo-root>\.claude\Docs\`
   - **Single file (HTML or MD only)**: save directly in the Docs folder
   - **Both formats**: create a subfolder `<topic>-<YYYY-MM-DD>/` and place both files inside it
   - Filename pattern: `<topic>-<YYYY-MM-DD>.html` or `<topic>-<YYYY-MM-DD>.md`
   - If a file with that name exists, append a counter: `<topic>-<YYYY-MM-DD>-2.html`

5. **Check for existing paired file (auto-sync)**:
   - Before writing, check if a paired file already exists (e.g., writing HTML — does a matching MD exist in the same folder or a paired subfolder?)
   - If a paired file exists, update it automatically with the same content so both stay in sync
   - If they were separate files before, move them into a shared subfolder

6. **Generate the file(s)** using the rules below

7. **Write the file(s)** and report the full path(s) back to the user

8. **After writing**:
   - For HTML: open in Chrome with `start chrome "<filepath>"`, then confirm
   - For Markdown: mention the path and that it's ready for future Claude context
   - For Both: open the HTML in Chrome, mention the MD path

---

## HTML Output — Dashboard Template

**ALWAYS read the template** at `C:\Users\NoahJackson\.claude\skills\doc-it\doc-template.html` before generating HTML. The template contains the full UI shell — sidebar, navigation JS, dark mode toggle, view-switching logic, print styles, and all CSS. You only need to fill in the placeholders.

### Placeholders

| Placeholder | Value |
|---|---|
| `{{TITLE}}` | Document title |
| `{{PROJECT}}` | Project name |
| `{{DATE}}` | Generation date (e.g., March 20, 2026) |
| `{{CONTENT}}` | The section content (see structure below) |

### Content Structure

The dashboard uses **view-switching** — only one section is visible at a time. The sidebar nav is auto-generated from the sections by JS.

Each top-level section is a `<section>` element. The JS reads `data-title` to build the nav and assigns color classes automatically.

```html
<section class="doc-section" data-title="Overview">
  <div class="section-card">
    <div class="section-card-header">
      <span class="section-card-num">1</span>
      <span class="section-card-title">Overview</span>
    </div>
    <!-- content: paragraphs, tables, lists, code, subsections -->
  </div>
</section>

<section class="doc-section" data-title="Data Flow">
  <div class="section-card">
    <div class="section-card-header">
      <span class="section-card-num">2</span>
      <span class="section-card-title">Data Flow</span>
    </div>
    <!-- content -->
  </div>
</section>
```

### Nested Components

**Subsections** (collapsible, within a section-card):
```html
<details class="subsection" open>
  <summary>Subsection Title</summary>
  <div class="subsection-body">
    ...content...
  </div>
</details>
```

**Step cards** (collapsible, numbered procedures — closed by default):
```html
<details class="step-card">
  <summary>
    <span class="step-num">1</span>
    <code>procedureName</code> — Description
  </summary>
  <div class="step-card-body">
    ...content...
  </div>
</details>
```

**Tables** (wrap for horizontal scroll):
```html
<div class="table-wrapper">
  <table>
    <thead><tr><th>Col</th><th>Col</th></tr></thead>
    <tbody><tr><td>Val</td><td>Val</td></tr></tbody>
  </table>
</div>
```

**Callouts**:
```html
<div class="callout callout-info">Info note</div>
<div class="callout callout-warn">Warning</div>
<div class="callout callout-danger">Critical</div>
```

**Badges**:
```html
<span class="badge badge-info">INFO</span>
<span class="badge badge-success">PASS</span>
<span class="badge badge-warn">WARNING</span>
<span class="badge badge-danger">FAIL</span>
```

### Content Conversion Rules

When converting markdown content to HTML:
- `# Heading` → a new `<section class="doc-section" data-title="...">` with a section-card inside
- `## Heading` → `<h2>` or a `<details class="subsection">` if grouping content
- `### Heading` → `<h3>`
- `#### Heading` → `<h4>`
- Bold, italic, lists, code blocks → their HTML equivalents
- Markdown tables → `<div class="table-wrapper"><table>...</table></div>` (th color inherited from parent section)
- Use `.callout-info` for notes/tips, `.callout-warn` for warnings, `.callout-danger` for critical
- Use `.badge` spans for status labels
- Gherkin/test scenarios → `<pre><code>` blocks
- Keep the content faithful to the original — do not summarize or truncate

### Dashboard Features (built into template)

- **Sidebar navigation**: auto-generated from `data-title` attributes
- **View-switching**: one section visible at a time, others hidden
- **Keyboard nav**: Arrow keys switch between sections
- **Dark mode**: toggle in sidebar footer, persisted to localStorage
- **Mobile responsive**: sidebar collapses to hamburger menu under 900px
- **Print**: sidebar hidden, all sections visible, clean layout
- **BWB branding**: Montserrat font, BWB color palette, embedded logo

---

## Markdown Output — AI-Optimized

Markdown files are structured for Claude to read as project context in future conversations. Prioritize machine-parseable structure over visual formatting.

### Format

```markdown
---
title: {{TITLE}}
project: {{PROJECT}}
generated: {{DATE}}
type: documentation
tags: [relevant, topic, tags]
---

# {{TITLE}}

## Section Name

Content here — concise, factual, structured.

### Subsection

- Key points as bullet lists
- Code references with file paths: `Services/OrderService.cs:42`
- Decisions and their rationale

## Section Name

| Column | Column | Column |
|--------|--------|--------|
| Data   | Data   | Data   |
```

### Markdown Rules

- YAML frontmatter with title, project, date, type, and tags (for future retrieval)
- Use heading hierarchy strictly: `#` for title, `##` for sections, `###` for subsections
- Prefer bullet lists over prose — Claude parses lists more efficiently
- Include file paths with line numbers where relevant
- Use fenced code blocks with language hints
- For tables, use standard GitHub-flavored Markdown
- Keep content faithful — do not summarize or truncate
- No collapsible sections or HTML in Markdown (keep it pure MD for AI parsing)
- Add a `## Key Takeaways` section at the end summarizing the most important points

---

## Auto-Sync Rules

When writing a file, check for its paired counterpart:

1. **Writing HTML, MD exists**: Re-read the HTML content and update the MD file to match (converting to AI-optimized format)
2. **Writing MD, HTML exists**: Re-read the MD content and update the HTML file to match (converting to dashboard format)
3. **Writing Both**: Generate both from the same source content in one pass
4. **Files in different locations**: If an HTML exists standalone and a paired MD is being created (or vice versa), move both into a shared subfolder: `<topic>-<YYYY-MM-DD>/`

The goal is that at any point in time, paired files contain the same information — one formatted for humans (HTML), one for AI (MD).

---

## Output Path

```
<repo-root>\.claude\Docs\<topic>-<YYYY-MM-DD>.html          (single HTML)
<repo-root>\.claude\Docs\<topic>-<YYYY-MM-DD>.md             (single MD)
<repo-root>\.claude\Docs\<topic>-<YYYY-MM-DD>\               (paired output)
  ├── <topic>-<YYYY-MM-DD>.html
  └── <topic>-<YYYY-MM-DD>.md
```

## Section Color Palette

The template supports 8 colors that cycle automatically (assigned by JS):
| Section | Color | Derived from |
|---------|-------|-------------|
| 1 | Blue (#0B69B1) | BWB Primary Blue |
| 2 | Teal (#2EA477) | BWB Teal Green |
| 3 | Steel (#4075AC) | BWB Secondary Blue |
| 4 | Green (#74C048) | BWB Primary Green |
| 5 | Ocean (#1a6b8a) | Blue-teal blend |
| 6 | Forest (#3d8b5e) | Green blend |
| 7 | Periwinkle (#5a7dba) | Blue blend |
| 8 | Sage (#4a9a6e) | Teal-green blend |

For documents with more than 8 sections, colors cycle back to section-1.