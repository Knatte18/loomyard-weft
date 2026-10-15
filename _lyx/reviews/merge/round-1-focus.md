---
round: 1
focus:
  - "Task's new engine usage readers (`internal/shuttleengine/transcriptread.go`, `claudeengine/usage.go`, `transcriptread.go`, `pricing.go`, `internal/shuttleengines/shuttleengines.go`) against the parent's `ModelChoice` embedding in `Spec`/`OpenSpec` and `Result.Engine`: does any task code build a `Spec`/`OpenSpec` literal, read `.Engine`, or resolve an engine name in a way the promoted-field form or the resolved-name rule no longer fits?"
  - "`PATTERN-model-choice-mapping` (new from the parent) against `tools/tokencount` (`count.go`, `calibrate.go`, `main.go`, `reads.go`) and the calibration path reading the Webster session engine: does any task code read `Resolved.Engine`, `Resolved.Model` or `Resolved.Params` outside `NewModelChoice`, or resolve a model key without returning a `ModelChoice`?"
  - "Parent's changed `requires.go` (`ErrRequirementsUnmet`, `ModelKey.Wrap`, generic way forward) against the task's engines: does `claudeengine` or the task's `shuttleengine` additions declare requirements or wrap refusals in a way that bypasses or contradicts the new contract?"
  - "`internal/shuttleengine/doc.go` and `PATTERN.md`'s changed `PATTERN-shuttle-provider-seam` line (both sides edited): does the task's new reader code keep provider specifics under `internal/shuttleengine/<engine>` and only `internal/shuttleengines` importing a concrete engine, and do the merged doc paragraphs still describe the merged code?"
  - "Parent's `internal/websterengine` role changes (`roles.go`, `runlevel.go`, `runwrites.go`) against the task's tokencount/calibration reads of Webster sessions and instruction files: does a renamed or reshaped symbol, role field or run-record key the task reads still match?"
  - "Test economy: `cmd/lyx/testdata/test-budget.yaml` for `internal/shuttleengine`, `internal/shuttleengine/claudeengine` and `internal/shuttlecli` after both sides raised counts: does the merged count match the tests the merge leaves, and did the parent's `enginedispatch_test.go`/`requires_test.go` additions make any task test redundant, or leave a merged behavior uncovered?"
---

Seeded from the overlap list and the parent's new invariants; each entry names the interaction, not a task-alone or parent-alone defect.
