---
round: 1
focus:
  - "internal/fabricengine/replay.go and internal/fabricengine/doc.go: the decision record's Scope lists only boardengine files; settle whether the records-replay change is licensed by the board entry (project files as rendered overviews) or is unlicensed scope, and whether it is correct for project-<slug>.md files."
  - "internal/boardengine/render.go, file naming and manifest: settle whether every project, open or done, gets project-<slug>.md, and whether RenderToDisk's manifest drops a file once its project is gone, with no stale file left."
  - "internal/boardengine/render.go, membership filtering: settle whether Tasks, Notes and Done in each file hold only entries of that file's project, section counts count only those entries, and layers still come from ComputeLayers over the whole board."
  - "internal/boardengine/render.go, Projects section: settle whether it lists only open projects directly under the file's project, linked with brief, ordered by parent's order key then handle (README by handle), with no lead line, member table or Standing column, and whether a done project is a linked Done row."
  - "internal/boardengine/render.go, After cell: settle whether it shows open dependencies' handles joined by a cell line break in depends_on order, omits done dependencies, and shows an unknown slug as a code span; check the empty and single-dependency cases."
  - "internal/boardengine/render.go, Notes ordering: settle whether research leads when configured, its heading is `Research` while other types keep the appended s, and a hub without a research type renders as before."
  - "Draft tasks: settle whether drafts get their own subsection after ready and layered tasks, never under Ready, and survive the per-project filtering."
  - "Project file header: settle whether the up link to the parent's file is right for root-level parents (README.md) and nested ones, and whether the beam/grouping line and effective owner are rendered per D2; check for broken or self-referential links."
  - "internal/boardengine/render_test.go and rename_test.go: settle whether each acceptance criterion D1 to D6 has coverage, whether any new test duplicates an existing one, and whether the rename_test.go change is licensed."
  - "internal/boardengine/doc.go: settle whether the `# README` section describes the new layout and carries no pinned perishable specifics; check the Render purity and declared-names constraints in render.go."
---

Seeded from the board entry and the decision record's D1 to D6 and acceptance criteria.
