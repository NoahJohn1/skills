---
name: doc-it
description: Outputs recent conversation content to a nicely formatted HTML or Markdown file for viewing documentation. Saves to ~/.claude/Documentation/<project-name>/ organized by project.
---

# Doc It

Convert the most recent documentation, analysis, comparison, or structured content from this conversation into a clean, readable file for archival and later viewing.

## Process

1. **Identify the content to export**:
   - Look at the most recent substantial output in the conversation (comparison docs, requirements, QA scenarios, analysis, summaries, etc.)
   - If ambiguous, use AskUserQuestion to ask which content to export
   - If the user specifies content in their prompt, use that

2. **Ask the user for output format**:
   - Use AskUserQuestion to ask whether they want **HTML** or **Markdown** output
   - HTML: styled, opens in a browser, good for presentation and sharing
   - Markdown: lightweight, portable, good for version control and editing

3. **Determine the project name**:
   - Derive from the current working directory (use the deepest meaningful folder name, e.g. `IndabaAPI`, `CareDaba`, `BuyBack`)
   - If at a generic path like `source/repos`, use AskUserQuestion to ask for the project name
   - Normalize to kebab-case for the folder name (e.g., `indaba-api`, `care-daba`)

4. **Check for existing project folder**:
   - Look in `C:\Users\NoahJackson\.claude\Documentation\` for a subfolder that resembles the project name
   - Use fuzzy matching (e.g. `indaba-api` matches `IndabaAPI` or `indaba-api`)
   - If a matching folder exists, use it; if not, create one

5. **Generate the file**:
   - Use the appropriate template below based on the chosen format
   - The filename should be descriptive of the content with a date suffix: `<topic>-<YYYY-MM-DD>.html` or `<topic>-<YYYY-MM-DD>.md`
   - If a file with that name already exists, append a counter: `<topic>-<YYYY-MM-DD>-2.html`

6. **Write the file** and report the full path back to the user

---

## Markdown Output Rules

When outputting as Markdown:
- Use standard GitHub-flavored Markdown
- Add a metadata header at the top:
  ```
  # {{TITLE}}
  > **Project:** {{PROJECT}} | **Generated:** {{DATE}}
  ---
  ```
- Preserve all original structure: headings, lists, tables, code blocks
- Keep the content faithful to the original -- do not summarize or truncate
- Use `> **Note:**` for callouts, `> **Warning:**` for warnings
- Use fenced code blocks with language hints where applicable

---

## HTML Template

Use this structure when HTML is chosen:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{{TITLE}}</title>
  <style>
    :root {
      --bg: #fdfdfd;
      --text: #1a1a2e;
      --muted: #555;
      --accent: #0B69B1;
      --accent-light: #D7EBFF;
      --border: #e0e0e0;
      --code-bg: #f5f5f5;
      --success: #74C048;
      --warn: #e8a735;
      --danger: #c0392b;
    }

    * { margin: 0; padding: 0; box-sizing: border-box; }

    body {
      font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
      color: var(--text);
      background: var(--bg);
      line-height: 1.7;
      max-width: 900px;
      margin: 0 auto;
      padding: 2rem 2.5rem 4rem;
    }

    header {
      border-bottom: 3px solid var(--accent);
      padding-bottom: 1rem;
      margin-bottom: 2rem;
    }

    header h1 {
      font-size: 1.8rem;
      font-weight: 700;
      color: var(--accent);
    }

    header .meta {
      font-size: 0.85rem;
      color: var(--muted);
      margin-top: 0.25rem;
    }

    h2 {
      font-size: 1.35rem;
      font-weight: 600;
      color: var(--text);
      margin: 2rem 0 0.75rem;
      padding-bottom: 0.3rem;
      border-bottom: 1px solid var(--border);
    }

    h3 {
      font-size: 1.1rem;
      font-weight: 600;
      margin: 1.5rem 0 0.5rem;
    }

    p { margin: 0.5rem 0; }

    ul, ol {
      margin: 0.5rem 0 0.5rem 1.5rem;
    }

    li { margin: 0.25rem 0; }

    table {
      width: 100%;
      border-collapse: collapse;
      margin: 1rem 0;
      font-size: 0.92rem;
    }

    th {
      background: var(--accent);
      color: white;
      text-align: left;
      padding: 0.6rem 0.75rem;
      font-weight: 600;
    }

    td {
      padding: 0.5rem 0.75rem;
      border-bottom: 1px solid var(--border);
    }

    tr:nth-child(even) td {
      background: #f9f9fb;
    }

    code {
      font-family: 'Cascadia Code', 'Fira Code', 'Consolas', monospace;
      background: var(--code-bg);
      padding: 0.15rem 0.4rem;
      border-radius: 3px;
      font-size: 0.88em;
    }

    pre {
      background: var(--code-bg);
      border: 1px solid var(--border);
      border-radius: 6px;
      padding: 1rem;
      overflow-x: auto;
      margin: 1rem 0;
    }

    pre code {
      background: none;
      padding: 0;
    }

    blockquote {
      border-left: 4px solid var(--accent-light);
      padding: 0.5rem 1rem;
      margin: 1rem 0;
      background: #f8fbff;
      color: var(--muted);
    }

    .badge {
      display: inline-block;
      padding: 0.15rem 0.5rem;
      border-radius: 3px;
      font-size: 0.78rem;
      font-weight: 600;
      text-transform: uppercase;
    }

    .badge-info { background: var(--accent-light); color: var(--accent); }
    .badge-success { background: #e8f5e1; color: #3a7d1a; }
    .badge-warn { background: #fff3d4; color: #8a6914; }
    .badge-danger { background: #fde8e8; color: #922; }

    .callout {
      border-radius: 6px;
      padding: 1rem 1.25rem;
      margin: 1rem 0;
    }

    .callout-info { background: #f0f7ff; border-left: 4px solid var(--accent); }
    .callout-warn { background: #fffbf0; border-left: 4px solid var(--warn); }
    .callout-danger { background: #fef5f5; border-left: 4px solid var(--danger); }

    footer {
      margin-top: 3rem;
      padding-top: 1rem;
      border-top: 1px solid var(--border);
      font-size: 0.8rem;
      color: var(--muted);
    }

    @media print {
      body { max-width: 100%; padding: 1rem; }
      header { border-bottom-width: 2px; }
    }
  </style>
</head>
<body>
  <header>
    <h1>{{TITLE}}</h1>
    <div class="meta">{{PROJECT}} &middot; Generated {{DATE}}</div>
  </header>

  <main>
    {{CONTENT}}
  </main>

  <footer>
    Generated by Claude Code &middot; {{DATE}}
  </footer>
</body>
</html>
```

## HTML Content Conversion Rules

When converting markdown content to HTML:
- `# Heading` becomes `<h2>` (h1 is reserved for the document title)
- `## Heading` becomes `<h3>`
- Bold, italic, lists, code blocks convert to their HTML equivalents
- Markdown tables become styled `<table>` elements
- Use `.callout-info` divs for notes/tips, `.callout-warn` for warnings, `.callout-danger` for critical items
- Use `.badge` spans for status labels (e.g., risk levels, pass/fail)
- Gherkin/test scenarios should go in `<pre><code>` blocks
- Keep the content faithful to the original -- do not summarize or truncate

---

## Output Path

```
C:\Users\NoahJackson\.claude\Documentation\<project-folder>\<topic>-<YYYY-MM-DD>.html
C:\Users\NoahJackson\.claude\Documentation\<project-folder>\<topic>-<YYYY-MM-DD>.md
```

## After Writing

- Report the full file path to the user
- For HTML: automatically open in Chrome with `start chrome "<filepath>"`, then confirm it was opened
- For Markdown: mention they can view it in any editor or preview tool
