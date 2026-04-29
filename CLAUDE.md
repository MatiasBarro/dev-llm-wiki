# dev-llm-wiki — Schema & Agent Instructions

A personal LLM-maintained knowledge base for software development research. The LLM owns the `wiki/` layer entirely — creating pages, updating cross-references, and keeping everything consistent. The user curates raw sources, directs analysis, and asks questions.

---

## Directory Layout

```
raw/                    ← immutable source articles (READ ONLY — never modify)
wiki/
  index.md              ← catalog of all wiki pages (update after every operation)
  log.md                ← append-only chronological log
  concepts/             ← fundamental ideas and definitions
  technologies/         ← tools, frameworks, databases, languages
  papers-articles/      ← per-source summary pages
  comparisons/          ← side-by-side analysis pages
CLAUDE.md               ← this file
mine/                   ← USER PERSONAL SPACE — NEVER TOUCH, READ, OR MODIFY
```

---

## Hard Constraints

- **`raw/`** — read-only. The LLM reads source files but never creates, edits, or deletes anything here.
- **`mine/`** — completely off-limits. Never read, write, or reference files in `mine/` for any reason.

---

## Page Format

**Filename**: kebab-case (e.g. `b-trees.md`, `apache-kafka.md`, `b-trees-vs-lsm-trees.md`)

**Required frontmatter** on every wiki page:

```yaml
---
type: concept | technology | paper-article | comparison
tags: [tag1, tag2]
sources:
  - "[[raw/Article Title]]"
date-updated: YYYY-MM-DD
---
```

**Cross-references**: Use standard Obsidian wikilinks — `[[page-name]]` for pages in the same folder, `[[concepts/b-trees]]` for cross-folder links.

---

## index.md Format

Organized by category. Update after every ingest or when a new page is created.

```markdown
# Wiki Index

## Concepts
| Page | Summary | Tags |
|---|---|---|
| [[concepts/b-trees]] | Balanced tree structure optimized for block storage | databases, storage |

## Technologies
| Page | Summary | Tags |
|---|---|---|

## Papers & Articles
| Page | Summary | Tags |
|---|---|---|

## Comparisons
| Page | Summary | Tags |
|---|---|---|
```

---

## log.md Format

Append-only. Each entry starts with a consistent prefix for easy parsing.

```
## [YYYY-MM-DD] <operation> | <description>
<1-2 line summary of what was done>
```

**Operations**: `init`, `ingest`, `query`, `lint`

**Parse tip**: `grep "^## \[" wiki/log.md | tail -10` shows the 10 most recent entries.

---

## Skills

| Command | Usage | Description |
|---|---|---|
| `/wiki:ingest` | `/wiki:ingest raw/<file> [--interactive\|--auto]` | Ingest a source; asks mode if no flag given |
| `/wiki:query` | `/wiki:query <question>` | Answer from wiki pages; offers to file the result |
| `/wiki:lint` | `/wiki:lint` | Health check: broken links, orphans, contradictions, gaps |

---

## Tagging Conventions

Use lowercase, hyphenated tags. Common tags for this domain:

- **Domain**: `system-design`, `databases`, `ai`, `api`, `distributed-systems`, `networking`, `security`
- **Type hint**: `algorithm`, `pattern`, `protocol`, `tool`, `framework`, `architecture`
- **Maturity**: `foundational`, `advanced`, `emerging`

---

## Naming Conventions

| Category | Folder | Example filename |
|---|---|---|
| Concept | `concepts/` | `cap-theorem.md`, `b-trees.md` |
| Technology | `technologies/` | `apache-kafka.md`, `postgresql.md` |
| Source summary | `papers-articles/` | `9-agentic-patterns-simply-explained.md` |
| Comparison | `comparisons/` | `b-trees-vs-lsm-trees.md` |
