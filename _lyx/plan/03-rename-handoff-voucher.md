# Card 3 — Rename the step clean-handoff marker to the handoff voucher

**Rename:**
- `internal/loomcli/selfreport.go` -> `internal/loomcli/entryobservation.go`
- `internal/loomcli/selfreport_test.go` -> `internal/loomcli/entryobservation_test.go`
- `internal/loomcli/stephandoff_test.go` -> `internal/loomcli/handoffvoucher_test.go`

**Edit:**
- `internal/loomengine/config.go`
- `internal/loomcli/arm.go`
- `internal/loomcli/halt_test.go`
- `internal/loomcli/step_test.go`
- `internal/loomcli/gotoentry_test.go`
- `internal/loomcli/smoke_bootstrapwiring_test.go`
- `internal/shedverbs/spec.go`

**Intent:**
Decision D4 makes the step clean-handoff marker a general handoff voucher with two writers, a completed `lyx loom step` and, from card 5, `lyx loom start`.
This card renames it, and the files that no longer hold self-report code, so the names stop saying "step" and "selfreport"; behavior does not change.

Follow the plan's Rename mechanic: `git mv` the three files first.
Then rename these identifiers and retarget every call site, in the renamed files and the Edit targets (`internal/shedverbs/spec.go` only in the comment that names `recordStepHandoff`):

- `loomengine.LoomStepHandoff` to `LoomHandoffVoucher`, returning `handoff-voucher.json`, and `LoomStepHandoffLock` to `LoomHandoffVoucherLock`, returning `handoff-voucher.json.lock`, in the same `.lyx` loom directory; a stale `step-handoff.json` is left on disk and read by nothing.
- `stepHandoffMarker` to `handoffVoucher`, `recordStepHandoff` to `recordHandoffVoucher`, `consumeStepHandoffMatch` to `consumeHandoffVoucher`.
- The test helpers `writeSelfreportStatus` to `writeStatusFixture` and `stepHandoffPaths` to `handoffVoucherPaths`, and the tests named after the marker (`TestConsumeStepHandoffMatch`, `TestObserveEntry_ConsumesStepHandoffIntoObservation`, `TestSmokeStep_RecordsCleanHandoffMarkerMatchingPersistedStatus`) to the voucher.

Update the doc comments of the renamed symbols and the two renamed source and test file headers to say "handoff voucher" where they said "step clean-handoff marker", keeping their content otherwise; the second writer is described by card 5, not here.

Run `grep -rn "StepHandoff\|stepHandoff\|step-handoff\|writeSelfreportStatus" --include=*.go .` afterwards; only `EntryObservation.CleanStepHandoff`, which card 4 renames, may still match.

**ImpactSummary:** identifier and file renames inside `loomcli` and `loomengine`, plus one comment in `shedverbs`; the voucher file's name on disk changes, its content and behavior do not.

**Commit:** 3: loom: rename the step clean-handoff marker to the handoff voucher
