---
round: 1
focus:
  - "internal/boardengine/sketch.go and layer.go: does `Board.MakeSketch` keep module labels, drop type, status and run fields in one write, and does the run lock (`runlock.go`) let only that change through and still refuse other body or scope edits?"
  - "internal/boardengine/layer.go, config.go and decision.go: the legacy `discussed` status migration. Does a task an older binary left in `discussed` become a Sketch at the next write without losing data, and is every remaining reference to the removed status, layer and README section gone?"
  - "internal/boardengine/config.go: `SketchType` is built in unless board.yaml lists it. Settle the handle letter `S` against collisions with board.yaml types, and what happens when board.yaml lists `sketch` with a different letter or definition."
  - "internal/loomcli/wiring.go: a finished recipe with no Finalize row calls `MakeSketch`. Does the call happen only for a Discussion-only (motif) run after the record is approved and stored, and is the entry left as a draft when the board refuses, with the failure reported?"
  - "internal/boardengine/render.go and internal/boardcli/text.go: do the README and `lym board list --text` list Sketches under their own heading as the Goal asks, with the ordering the commit message claims (first among Notes in the README, last in the text list)?"
  - "internal/battencli/startguard.go: why this file changed, and whether the change is licensed by the Goal. Check that a Sketch can never be started as a run."
  - "Docs the change touches (docs/overview.md, plugins/ly/skills/board/SKILL.md, boardengine/doc.go, boardcli/doc.go, loomcli/doc.go): do they match the code as the diff leaves it, and does the repository's constraints document name a doc this diff should also have changed?"
  - "Test economy across sketch_test.go, config_test.go, render_test.go, text_test.go, verbs_test.go and the loomcli tests: new tests that duplicate existing coverage, and added behavior with no test."
---

The Goal in the decision record is the only measuring stick; its other sections are empty.
The commit message is the writer's claim, to be checked against the diff.
