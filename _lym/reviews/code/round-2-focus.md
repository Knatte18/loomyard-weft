---
round: 2
focus:
  - "internal/fabriccli/clone.go `CloneAndWire` and its comment near the made-event call: does a test that installs the real `heddleengine.Seam` and clones show the prime's anchor holding rendered heddle files, and do the comment and `internal/fabricengine/doc.go` claim only what that test proves?"
  - "internal/fabriccli/clone.go `CloneAndWire`, the `hubreconcile.Ensure` error path: when the walk fails after a failed render, does the returned error carry the seam warning text, and does the comment state that the prime is rendered only when the render succeeded?"
  - "internal/heddleengine/seam.go `Seam.render`: when a made event's render writes none of its targets because of `Report.Warnings` (blocked temp dir, target outside the exclude list, unmarked existing file), does it return an error naming the warnings, and does heddleengine's seam test cover that row?"
  - "internal/fabricengine/worktreeseam_integration_test.go clone made-event subtest: does it assert, at the moment the made event fires, that the module-config commit is in the weft history and the hub walk has not run, and does a walk-failure row assert the event was heard before the clone failed?"
  - "internal/fabriccli/clone.go envelope map with `seam_warning`: is there a test running the clone verb through `runCloneWithReset` that asserts the field names `lym heddle render` for a failing seam and is an empty string for a healthy one?"
---

The entries name the sites the round-1 review found weak; the reviewer settles each question from the diff.
