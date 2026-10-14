# Webster: split batches on renames, run guard tests per card

Webster's checks now agree with the code a plan changes.
Two failures closed: a correct batch was rejected when a later card renamed or deleted what an earlier card of the same batch targets (tender batch 07), and a card's gate missed repo-scanning tests, so a cross-package break surfaced only at a later gate (#594, tender batch 01).

## What changes

- **Batch boundary.**
  A card whose Rename old side or Delete target covers a target of an earlier, distinct card in its candidate batch now starts a new batch.
  The earlier batch is then done-checked and gated against a tree where its targets still stand, so its glyphs resolve and its gate names directories that exist.
  `done-checks`, bind, drift, the scope guard and `gateScopeOf` are unchanged.
- **Guard step per card.**
  Every card gate ends, before the two lints, with one `lyx gate test --tags integration <guard packages> -- -run '^(<names>)$'` step.
  It runs every `//lyx:guard` test of the root module, so a break such as `TestGitrepoBoundary_PinnedRunCallSites` or `TestTierPurity_UntaggedTestsSpawnNothing` fails at the card that causes it.
  `begin-batch` and `recover-batch` render the same step.
- **Fork prompt.**
  A guard failure may require a fix in a file the card does not list; the fork fixes it under the card's commit, preferring a code fix over moving the pin.
  The wording lives in the webster stencils.

## Design decisions

- **Covers.**
  `planparser` gains the ref-covers logic (`RefScope`/`RefFile` based, never local string tests).
  A symbol-level ref covers the same member; a file-level ref covers that file, glyphs whose unit is that file, and glyphs or `plan:` handles whose unit is the file's package directory; a directory-level ref covers anything at or under it.
  A ref `planparser` cannot map never covers and is never covered.
  `Uses` is excluded, and a card never splits from itself.
- **One batcher helper.**
  `internal/batcher` holds the rule; the cost batchifier treats a segment holding such a pair as infeasible (a one-card segment stays feasible), and the identity batchifier is unaffected.
  It applies at first init and at rebaseline.
  A partition already recorded in `state.json` is never regrouped.
  The file-level rule over-splits rather than under-splits: it splits from an earlier card whose glyph unit is that file's package directory even when the member lives in another file.
- **Shared guard scan.**
  `internal/impactset` exports its marker scan and command-length limit; webster passes its own walked directory list.
  The walk matches `go list ./...`: it skips `testdata`, `vendor`, directories starting with `_` or `.`, and anything under a nested `go.mod`, and spawns no process.
  The impactset round gate's guard tests are unchanged.
- **Emptied guard directories.**
  A guard directory drops from the step for the card that renames out or deletes all its Go files, and for later cards in the batch; earlier cards and a partial move keep it.
- **Scan failure.**
  A `GuardScanError` or an over-long step renders no guard step and puts a line in the fork prompt (the plan-level verify still runs the guard tests); `begin-batch` does not refuse.
  A read error in the walk refuses `begin-batch` and `recover-batch`, with a new transient row in `contracts/specs/refusal-spec.md`.
- Docs updated: `doc.go` of `batcher`, `websterengine` and `impactset`, and `contracts/specs/loom-plan-spec.md` where it describes batching and the per-card gate.

## Known limits

- Each guard step adds a gate-slot acquire and a few seconds per card; a slow guard test, or one failing on a parent break, fails every card gate.
- Coverage is as good as the `//lyx:guard` marking; unmarked pins, and guard tests an earlier card of the same batch adds, are caught only by the batch gate or plan-level verify.
- Go only: a .NET hub gets no guard step.
- A persistent `GuardScanError` turns the guard step off for later batches until the file is fixed.
- A fork may loosen a guard's pinned set to pass; the edit shows as a scope-guard deviation.

## To check by hand

- `TestGofmt_ModuleFilesAreFormatted` failed once on `internal/planparser/targetdirs.go:149` (`||strings` spacing); commit `66e1f767d` fixed it, and the fixer reported `lyx gate test --tags integration ./cmd/lyx ./internal/planparser` green.
  Confirm with a full `lyx gate test ./...` and `lyx gate test --tags integration ./...`.
- The scope guard reported these files as changed outside the batch's own targets: `internal/batcher/boundary.go`, `internal/batcher/cost_test.go`, `internal/planparser/targetdirs.go`, `internal/websterengine/beginbatch_test.go`, `internal/websterengine/guardstep.go`, `internal/websterengine/guardstep_test.go`.
- The test file `internal/websterengine/cardgate_test.go` was extended by a shell append rather than an editor; review that addition for formatting and placement.
- Run a tender-batch-07-shaped plan end to end: the rename card should land in a later batch than the creating cards, and both batches should pass record-batch.

Closes #594
