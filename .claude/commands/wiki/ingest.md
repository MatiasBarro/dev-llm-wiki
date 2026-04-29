---
name: wiki:ingest
description: Ingest a raw source into the wiki — interactive (discuss first) or automated (process end-to-end)
argument-hint: "<source-file> [--interactive | --auto]"
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Grep
  - Glob
---

<objective>
Ingest a raw source file from raw/ into the wiki. Two modes:
- **Interactive**: discuss key takeaways with the user before writing anything
- **Automated**: process the source end-to-end without stopping

If neither --interactive nor --auto is specified in the arguments, ask the user which mode they want before proceeding.
</objective>

<constraints>
- raw/ is READ ONLY — never create, edit, or delete anything inside raw/
- mine/ is COMPLETELY OFF-LIMITS — never read, write, modify, or reference it
- All wiki pages go in wiki/ under one of: concepts/, technologies/, papers-articles/, comparisons/
</constraints>

<page-format>
Every wiki page requires this frontmatter:
```yaml
---
type: concept | technology | paper-article | comparison
tags: [tag1, tag2]
sources:
  - "[[raw/Article Title]]"
date-updated: YYYY-MM-DD
---
```
Filenames: kebab-case (e.g. `b-trees.md`, `apache-kafka.md`)
Cross-references: Obsidian wikilinks `[[page-name]]` or `[[concepts/page-name]]` for cross-folder links
</page-format>

<context>
Arguments: $ARGUMENTS
</context>

<process>
## Step 0 — Determine mode

If $ARGUMENTS contains `--interactive`, use interactive mode.
If $ARGUMENTS contains `--auto`, use automated mode.
If neither flag is present, ask the user: "Interactive (discuss key takeaways first) or automated (process end-to-end)?"

---

## Interactive mode

1. Read the raw source file in full
2. Read `wiki/index.md` to understand what's already in the wiki
3. Present **3–5 key takeaways** — what's most important, surprising, or worth emphasizing; note connections to existing wiki pages
4. Discuss with the user: clarify what to emphasize, what to skip, what's already covered
5. Based on the discussion, identify which wiki pages to create or update
6. Write or update each page with proper frontmatter, body content, and cross-links between pages
7. Update `wiki/index.md` — add new pages to the correct category table
8. Append to `wiki/log.md`: `## [YYYY-MM-DD] ingest | <source title>`
9. Report: files created, files updated, cross-links added

---

## Automated mode

1. Read the raw source file in full
2. Read `wiki/index.md` to understand existing wiki pages and avoid duplication
3. Identify: categories touched, concepts/technologies/comparisons covered
4. Create a summary page in `wiki/papers-articles/` with key takeaways and cross-links
5. Create or update relevant pages in `concepts/`, `technologies/`, `comparisons/`
6. Add cross-links between all touched pages
7. Update `wiki/index.md` with all new pages
8. Append to `wiki/log.md`: `## [YYYY-MM-DD] ingest | <source title>`
9. Report all files created or updated with a brief description of what changed
</process>
