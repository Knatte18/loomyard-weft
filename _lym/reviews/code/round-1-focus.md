---
round: 1
focus:
  - "internal/boardengine/board.go and store.go: does promote or any status release now write depends_on in the same write that releases a task, so dobby never sees a released task with empty deps?"
  - "internal/boardcli/cli.go: does set-deps, set-status and promote ordering match the Goal's Expected line (start only when every depends_on entry is done or gone, raddle included)?"
  - "internal/boardengine/sketch_test.go TestDiscussedPromotion: does it cover the raddle-recipe dependency case, and does it overlap other tests?"
  - "internal/boardcli/doc.go and internal/boardengine/doc.go: do the doc changes match the code behavior after the diff?"
  - "Commit messages in the range: any Pattern-Departure trailers, and do they name real entries?"
---

The decision record carries only the board entry under `## Goal`; the other sections are empty, so the entry is the measuring stick.
