---
name: wiki:query
description: Query the wiki — find relevant pages and synthesize an answer with wikilink citations
argument-hint: "<question>"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Grep
  - Glob
---

<objective>
Answer a question using the wiki as the primary knowledge source. Synthesize from existing wiki pages with inline wikilink citations. After answering, offer to file the answer as a new wiki page so the insight compounds in the knowledge base.

Never modify raw/ or mine/.
</objective>

<constraints>
- mine/ is COMPLETELY OFF-LIMITS — never read, write, or reference it
- raw/ is READ ONLY — read source files only as a fallback when the wiki doesn't cover the topic yet
- New wiki pages go in wiki/ under concepts/, technologies/, papers-articles/, or comparisons/
</constraints>

<page-format>
If filing a new wiki page, use this frontmatter:
```yaml
---
type: concept | technology | paper-article | comparison
tags: [tag1, tag2]
sources:
  - "[[wiki/concepts/related-page]]"
date-updated: YYYY-MM-DD
---
```
Filenames: kebab-case. Cross-references: `[[page-name]]` or `[[concepts/page-name]]`.
</page-format>

<context>
Question: $ARGUMENTS
</context>

<process>
1. Read `wiki/index.md` to identify which wiki pages are relevant to the question
2. Read those pages in full
3. If the wiki doesn't cover the topic yet, fall back to reading relevant files in `raw/`
4. Synthesize a clear, direct answer with inline `[[wikilinks]]` citing the source pages
5. After delivering the answer, ask: **"Want me to save this as a wiki page?"**
   - **Yes**: ask for a title and category (concepts / technologies / papers-articles / comparisons)
     - Create the page with proper frontmatter
     - Update `wiki/index.md` to add the new entry
     - Append to `wiki/log.md`: `## [YYYY-MM-DD] query | <page title>`
   - **No**: leave the answer in chat only
</process>
