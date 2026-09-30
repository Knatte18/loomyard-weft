---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    location: internal/shuttleengine/startup_test.go:768
    summary: the doc comment of TestStartup_RunDeadlineShorterThanWindow_NeverReady still reads NeverReadypins, the space card 4 dropped is not restored
  - id: F2
    severity: NIT
    location: internal/shedengine/run_commitstatus_test.go:207
    summary: card 1 reformats two table rows of a test file its card does not name, a cosmetic out-of-card edit that re-aligns their name fields
---

Review range: `product.parent` is `main` (from `_lyx/shed/ly-drive-safe-repairs/status.json`); the subject is `git diff $(git merge-base main HEAD)..HEAD`, merge base `676d9ec03`, HEAD `38cb96779`.

Substrate: `go build ./...` passes; `go vet` and `go test` over `internal/shuttleengine` and `internal/shedengine` pass; `gofmt -l` over every changed Go file prints nothing.

### [LOW] NeverReady doc comment still misses its space

**Location:** internal/shuttleengine/startup_test.go:768
**Issue:** The line reads `// TestStartup_RunDeadlineShorterThanWindow_NeverReadypins the run-deadline-anchoring decision: a`.
At the merge base it reads `..._NeverReady pins ...`.
Card 4 (`3e5420f01`) dropped the space, and no later commit restores it.
Apart from the space, the line matches the merge base.
The doc comment no longer starts with the function name followed by a space, and gofmt cannot catch a change inside a comment.
**Fix:** Restore the space so the line is byte-identical to the merge base.

### [NIT] Out-of-card reformat in run_commitstatus_test.go

**Location:** internal/shedengine/run_commitstatus_test.go:207
**Issue:** Card 1's Edit list names `shed.go`, `status.go` and `run.go`, not this file.
Commit `f1b6466ac` still splits the two one-line `funcProducer` literals in `TestRun_PersistFailureOnAFailedTerminalSurfacesBothErrors` across three lines each.
gofmt then re-aligns the `name:` keys, which is why the whole-range sweep lists them as whitespace-only pairs.
The test's behaviour is unchanged, and the original was gofmt-clean.
It is unrelated churn in a file no card asks to touch.
**Fix:** Restore the file to its merge-base content; no later commit touches it.

## Focus directives settled

- `startup_test.go:768`: not restored; the line differs from the merge base only by the missing space (F1).
- Whole-range whitespace sweep: I paired every removed line with every added line in each hunk of `git diff 676d9ec03..HEAD`, comparing them with all whitespace stripped.
  Twelve pairs differ only in whitespace.
  Nine are gofmt re-alignments of struct fields forced by fields the cards add: `shedengine.Status` gains `Transient` plus a two-line comment, which splits the alignment block, and `shedverbs.LastStep` gains `BinaryChanged`, which widens it.
  These are not defects.
  Two are gofmt re-alignments forced by the out-of-card reformat in F2.
  The last one is F1.

## Other checks that found no defect

- Files touched outside every card's Create/Edit list: `internal/landingshed/ctx.go` gains `transientFailure`, the shared helper that implements card 8's rule (cancellation first, then `shedtransient.Class`) for both `publish.go` and `finalize.go`.
  It serves card 8's intent directly, so it is not a defect.
  The other file is F2.
- Per-card mechanical check: every card group is `Create` or `Edit`, so no `Rename` AST script or `Delete` `assert-no-callers` check applies.
