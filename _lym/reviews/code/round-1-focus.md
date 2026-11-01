---
round: 1
focus:
  - "`internal/boardengine/render.go` `Render`: `project-root.md` is keyed by `projectFileName(\"root\")`. Settle whether a real project with slug `root` (or a design or other output file) can collide with or overwrite it, and whether slug validation or the Bolt replayer's `project-` prefix handling treats the new file correctly."
  - "`projectSection` member count and `projectBody` for the root: settle that the root's section and file hold its own entries alone, that open projects under no other still appear exactly once in README.md (as `##` sections, not also as rows), and that an open project under a done project is still placed correctly now that the root's children are rendered by `renderReadme` rather than by `projectBody`."
  - "`entrySections(rootProject, ...)` and its Done table: settle that a done project under no other, and root's done tasks and notes, land in `project-root.md` and README.md's root section with correct links, counts in headings and empty-section lines, and that no row is listed in both the root section and a project section."
  - "`fileHeader` and `upLink` for the root and for top-level projects: settle that `project-root.md` links up to README.md, that a project whose parent is not on the board does too, and that the root's facts line (`Owner` and prime owner, no slug, no age, no design link) matches the decision record's heading `P0 Root: entries under no project`."
  - "Heading levels in README.md after the `Prime` wrapper is gone: settle that every project section is `##`, the root's is last, `Own entries` placement and the six-level cap still hold, and that no other code (grep for `Prime`, `No project`, `primeSection`, `noProjectSection`, `fileLink`, and the README's header or intro text in other packages, docs and skills) still depends on the removed names or layout."
  - "Test economy in `render_test.go` and goldens: settle that the changed tests cover the new root layout (README last section, `project-root.md` content, root with and without open sub-projects), that no new test duplicates an existing one's coverage, and that each behavior the diff adds has coverage."
  - "Claims in `docs/overview.md` and `doc.go` against the diff: settle that the stated layout, the heading `P0 Root: entries under no project`, the facts line and the `Own entries` rule match what `render.go` does, and that the decision record's removal of `Prime`, `No project` and a duplicated `Own entries` heading is fully landed."
---

Seed focus for round 1, set from the decision record's Goal and the single commit in the range.
