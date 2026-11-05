```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: challenge the F1, F2 and F3 proposals.

- **LOW: F2's added row does not exercise the `now.Status == nil` clause.**
  - Changed text: `sketch_test.go`, row `SketchThatKeepsItsRunStatusIsRefused`, and the reviewer's claim that dropping the clause lets a held note through.
  - Evidence: `internal/boardengine/runlock.go:152-156`. When `endsAsSketch` is false, the code falls to `!kept || scopeChanged(before, now, removedDone)`.
  - For this row, `before` is a task with Recipe `motif` and labels `bug, board`. `now` is a note with labels `sketch, board` and Recipe `""`. Kind, Labels and Recipe all differ, so `scopeChanged` is true and `RunLockedError` is returned.
  - With the clause removed, `endsAsSketch` would be true only if `!scopeChanged(sketchOf(before), now)`. In this row that comparison holds, so the exemption would pass.
  - The case where dropping the clause leaks is therefore a mutation where `endsAsSketch` returns true and short-circuits at line 152. That is exactly what this row triggers.
  - The suite would fail on this row under that mutation, so the row does pin the clause.

  I withdraw this finding. The short-circuit at line 152 happens before the line 155 fallthrough, so the row is a valid regression guard. No finding.

- **LOW: F1's `got.Revised.Before(created)` assertion cannot detect a missing Revised stamp.**
  - Changed text: `sketch_test.go` `HeldTaskBecomesASketch`, `!got.Created.Equal(created) || got.Revised.Before(created)`, and the `doc.go` and `summary.md` text "the write stamps Revised like any scope change".
  - Evidence: Created and Revised are both whole-second UTC. A Revised that was never restamped still equals or follows Created, because `dates.go:35-39` sets both to now for a new entry. The assertion passes with or without the stamp, so the documented claim is unpinned.
  - Pinning it needs a Revised that provably differs from the pre-write value, which a sub-second test cannot get without an injectable clock. The doc wording is accurate per `dates.go:48-49`; only the test is weak.

- **F3: no finding.** The deleted bullet's premise is false: `commitstatus_integration_test.go:52` has the `RefusedSketchLeavesTheEntryDraft` row (depended-on task, `StatusDraft`, refused-sketch flag true).

Tool calls:
1. Ran grep on `runlock.go` for `endsAsSketch` and on `internal/loomcli` for the refused-sketch row. My context lacked the run-lock call site and the loomcli test.
