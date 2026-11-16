---
round: 1
focus:
  - "internal/boardengine/render.go runFieldLines: the board entry asks for every per-task field that changes how the run goes and that the cell omits, and to check the Task field list for others. Settle whether the diff's choice of pull_request and on_land, with profile and recipe left to the Recipe column, covers the Task fields in task.go that change a run (for example notify_on_land, isolated, owner_machine, front), or whether a field of that kind is missing."
  - "internal/boardengine/render.go entryCell: settle whether on_land is shown for entries where it is meaningful, given task.go says only a front task carries it, and whether the append(runFieldLines(t), settingLines(...)...) order and the codeSpan(cellText(...)) escaping keep a pipe or special character in a value from breaking the table row."
  - "internal/boardengine/render_test.go TestRenderReadmeGolden and readmeFixture: settle whether the golden rows pin the bullet order (brief, pull_request, on_land, settings) and the empty-field case, whether the fixture edits break any other golden test that shares readmeFixture, and whether the added test-comment lines state a pin the assertions make."
  - "internal/boardengine/doc.go README entry-cell paragraph and the commit message: settle whether each claim (field order, code-span form, 'run field the entry sets', Recipe and profile getting no bullet) matches what render.go does, and whether any other doc that describes the entry cell (specs, README docs) still says the cell holds only the brief and run settings."
---

Seed pass. The decision record holds only the board entry under `## Goal`, so the amended entry is the measuring stick.
The diff touches three files: `doc.go`, `render.go` and `render_test.go`.
