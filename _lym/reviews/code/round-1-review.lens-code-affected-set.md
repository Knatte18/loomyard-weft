```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set. Review what the single commit `5655ca77` forces outside the lines it touches.

**Per change, the set I found and whether the diff covers it**

1. Removed `projectsSection`, `projectSections` and `projectsHeading`: a repo-wide grep finds no remaining reference outside `_lym` and `.lym`. Covered.
2. Removed Projects bullet list and flat member table (output shape): the changes in `render.go` (header), `doc.go` (lines 169–181) and `docs/overview.md:364` are updated. One stale sentence remains, filed as F1. Covered, except F1.
3. `countedHeading` signature changed from a string level to an int level: the only callers are in `render.go` (`projectSection`, `entrySections`). No test or other package calls it. Covered.
4. `entrySections` signature changed from a string level to an int level: the callers are `projectBody` and nothing else. Covered.
5. `columnEntry`, `entryCell` and `emptySectionLine` are still used by the Done table, `waitTable`, `noteTable` and the empty Ready line, so none is dead. The `priority` option on `readmeTable` is still used by `noteTable`. Covered.
6. README layout (Prime at level 2, sub-projects first, own entries last) and project-file layout: `doc.go:172–181` and `docs/overview.md:364` match what `renderReadme` and `renderProjectFile` produce. `render_test.go` goldens are updated. Covered.
7. Stencils, skills and other docs that describe the README: a grep for "one section per open project", "Projects section", "member table" and "Prio column" finds none outside `doc.go` and `overview.md`. Covered.
8. PATTERN.md: no entry covers board render layout, and the commit declares no `Pattern-Departure`. Nothing to land.
9. Commit message: it matches the diff. The `Co-Authored-By` trailer names Opus 5.5 while this session's attribution says Sonnet 5.5; that is not a repo-rule finding.

**Findings**

- **F1, LOW, consistency**: `internal/boardengine/doc.go:75`.
  - The sentence reads "The README shows a note's priority as H or L in the Prio column of the Notes tables and the project sections".
  - This is stale. The `projectSections` member table with the Prio column is removed, and project rows no longer carry Prio.
  - Only the Notes tables carry it now. The same commit fixed `doc.go:193` to "a Notes row adds the Prio column", so the two sentences disagree.
  - Fix: change line 75 to "...in the Prio column of the Notes tables, and no view orders by it."

**Tool calls (2 of 6)**
1. A repo-wide grep for stale prose terms (Projects section, member table, Prio column, project sections), to find docs, stencils and tests that describe the old layout. My context only held the diff and `render.go`.
2. A grep of `doc.go` for README, project and prime wording, plus the removed symbol names repo-wide, to check for leftover references in places my context had not read.
