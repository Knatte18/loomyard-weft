---
round: 1
focus:
  - "internal/beamcli/mergeparent.go `freeMergeTaskSlug`: does it skip every slug a board entry, a run directory or a task branch holds, including the truncated beam slug plus the `-merge-in-N` tail, and does the loop always terminate?"
  - "internal/beamcli/deps.go `HeldSlugs`: does it return both run directory names and task branch slugs with prefix \"\", and does an error from either source refuse the verb with a way forward instead of falling back to a taken slug?"
  - "internal/shedrun/paths.go `RunIDs` and its use in internal/adoptcli/fixloop.go: is the refactor behaviour-preserving against the removed inline reader (missing directory, non-directory entries, error path), and is the directory listed the one `RunDir` and `runSegment` resolve, legacy `self` alias included?"
  - "Scope against the Goal in the decision record: does the diff change only what the board entry asks (a free slug for a second merge-in), and is every change to `fixloop.go`, `land.go` and `cli.go` licensed by that or by the shared helper it needs?"
  - "Tests: does `TestMergeParent` cover a held run directory, a held task branch and the numbering past -2, and does the `RunIDs` test pin behaviour the callers rely on without duplicating `TestMergeParent`?"
  - "Docs: do the `merge-parent` help text in `cli.go`, `doc.go` and the `landDeps` comment agree with the code, and does any other doc naming the merge task slug rule need to land in this diff?"
---

Seed focus for round 1. The decision record carries only the imported Goal, so the board entry is the measuring stick.
