```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

- **Removed symbols:** nothing outside `internal/driftengine` uses `ScopePartial`, `Acceptance` and its consts, the dropped `ScopeRecord` fields, or the 3-arg `DiffCaptures` signature. Module-wide vet was refused, so the grep is my substitute. The fixer's build, vet and `lym gate test` over `driftengine`, `batcher` and `configsync` already passed.
- **F3 behaviour changes:**
  - `scopeText` now answers `full.` for any non-off scope, so a stored `partial` renders as `full.`.
  - `classifyCurrent` and `settleUnchangeable` now diff with no leave-out list.
  - `renderVerdict`'s default branch renders `- Check: not run.`, which `NoCheckSaysNoneRan` still pins.
  - The tolerant-read row in `recordsguard_test.go` uses `{"scope":"full","skip":[...],"accepted":[...]}` and passed, so unknown keys are ignored.
- **F10 test:** the row asserts `or run "lym config reconcile --layer default --apply"`. The new message contains that string and the old `lym fabric reconcile` text does not, so the row fails with the defect in place.
- **F11 row (traced, not run):**
  - With the `!fileAbsent` clause dropped, `FollowTemplate` would receive the full-template base as present.
  - It would then treat each template alias absent from the prime copy as an operator deletion, so `Added` would lack `sonnet` and the row would fail.
  - The row passes under the current code.
  - Whether `Added` lists the top-level path `sonnet` was confirmed only by that passing run.

Calls made:
1. `go vet ./...` — refused by the guard, nothing ran.
2. `grep` for the removed symbols across `internal`, `cmd` and `tools` — to check callers outside `driftengine`, which the fixer's three-package build does not cover.
