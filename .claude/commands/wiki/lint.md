---
name: wiki:lint
description: Health-check the wiki — find orphan pages, broken links, contradictions, missing pages, and content gaps
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Grep
  - Glob
---

<objective>
Run a full health check on the wiki. Surface structural problems (broken links, orphans) and content problems (contradictions, missing pages, stale claims). Present a prioritized report and offer to fix selected issues immediately.

Never modify raw/ or mine/.
</objective>

<constraints>
- mine/ is COMPLETELY OFF-LIMITS — never read, write, or reference it
- raw/ is READ ONLY
</constraints>

<process>
1. Read `wiki/index.md` for the full page inventory
2. Read all wiki pages across `concepts/`, `technologies/`, `papers-articles/`, `comparisons/`
3. Run these checks:

**Broken** (highest priority):
- Every page listed in `index.md` exists on disk
- Every page has valid frontmatter: `type`, `tags`, `sources`, `date-updated` fields present
- Every `[[wikilink]]` in page bodies resolves to an existing file in `wiki/`

**Missing**:
- Concepts or technologies mentioned by name in page bodies but with no dedicated wiki page

**Orphaned**:
- Wiki pages that have no inbound `[[wikilinks]]` from any other wiki page

**Suggestions**:
- Questions worth asking that the wiki can't yet answer well
- Topics where only one source covers the claim (fragile — worth finding a second source)
- Content gaps visible from the index (categories that are thin or missing)

4. Present findings grouped by severity: **Broken → Missing → Orphaned → Suggestions**
5. Ask: "Which of these would you like me to fix now?"
   - Fix selected issues (create missing pages, repair broken links, add cross-links)
   - Update `wiki/index.md` as needed
   - Append to `wiki/log.md`: `## [YYYY-MM-DD] lint | <summary: N broken, N missing, N orphaned>`
</process>
