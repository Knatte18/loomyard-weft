---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/01-remove-tier1-filing.md
    summary: Card 1 deletes LoomSelfreportFiledLock but never names the halt_test.go newHaltFixture directory loop that calls it, so the card's tree does not build as written
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/06-agent-filed-issues-docs.md
    summary: Card 6 Intent states the reflection drop rule but not what it can let through or what bounds it, which D5 and the attack-surface question require
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/plan/04-anomaly-friction-notes.md
    summary: Card 1 deletes the surviving voucher step test and its handoffSteps helper, and card 4 re-adds the same step-level test and voucher rows that TestConsumeStepHandoffMatch and TestObserveEntry_ConsumesStepHandoffIntoObservation already cover
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/plan/05-start-vouches.md
    summary: Card 5 Intent states that the live-strand no-op writes no voucher and that each write replaces an earlier voucher, but its test list pins neither, and it does not say the one-shot and mismatch bounds are pinned by the renamed existing tests
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/03-rename-handoff-voucher.md
    summary: Card 3 renames the identifiers but its closing grep cannot catch the prose "step clean-handoff marker" left in arm.go, halt_test.go and smoke comments, and the Intent names only the renamed symbols' and files' comments
---

### [MEDIUM] Card 1 build break at the halt fixture's directory loop

**Location:** `_lyx/plan/01-remove-tier1-filing.md`, Edit/Delete lists and the `halt_test.go` bullet.
**Issue:** `internal/loomcli/halt_test.go:107` builds the fixture's directories from `loomengine.LoomSelfreportFiledLock(loc)` as well as `LoomStepHandoffLock(loc)`.
Card 1 deletes `LoomSelfreportFiledLock`, and the `halt_test.go` bullet lists the `fileIssue` fake, `filed`, `tierOneOn` and the `TierOne` tests but not this loop, nor the comment above it about the filed-title marker.
Card 1's tree therefore fails to compile at `newHaltFixture` until the implementer finds it.
The `Selfreport: false` literal on line 117 is card 2's, and `f.c.entryObservation = crashResumeEntry()` at line 584 sits inside a `TierOne` test that card 1 deletes, so those two are fine.
**Fix:** add a `halt_test.go` bullet to card 1: drop `LoomSelfreportFiledLock` from the fixture's directory loop and reword the comment above it.

### [MEDIUM] Card 6 states the drop rule without its bound

**Location:** `_lyx/plan/06-agent-filed-issues-docs.md`, `**Intent:**`, the stencil bullet.
**Issue:** D5 replaces a deterministic filer with agent judgement and names the bound: the rule can leave a real lyx fault behind a halt unfiled, and the evidence stays (the note names the trace file, status and history stay on disk, `lyx selfreport create` remains available).
Card 6's Intent gives the rule and the "process signal is not a defect" sentence, but no sentence says what the rule can let through or what bounds it.
The attack-surface question puts that statement in the card that introduces the change.
**Fix:** add the bound to the stencil bullet's Intent, and have the stencil keep the `lyx selfreport create` pointer so the operator path is named where the drop happens.

### [MEDIUM] Card 4's new tests overlap existing voucher tests, and card 1 deletes the one that survives

**Location:** `_lyx/plan/04-anomaly-friction-notes.md`, Tests, the `halt_test.go` step-level bullet and the no-note table; `_lyx/plan/01-remove-tier1-filing.md`, the `halt_test.go` bullet.
**Issue:** `TestStep_TierOne_HandoffMarkerSuppressesCrashResume` with its `handoffSteps` helper already drives two steps through `stepOver` with a real observing `PreStep`; its `MarkerKept` and `MarkerRemoved` subtests are card 4's "completed step then second step does not write" and "no voucher writes a note" rows.
Card 1 deletes it as a `TierOne` test and card 4 writes it again.
`TestConsumeStepHandoffMatch` pins match, history growth, state mismatch and absent voucher with delete-on-read, and `TestObserveEntry_ConsumesStepHandoffIntoObservation` pins the one-shot second call, so card 4's "voucher matches" row and "second observation writes" case are covered below the note writer.
`PATTERN-test-economy` asks for an existing test to be extended, not deleted and re-created.
**Fix:** card 1 keeps `handoffSteps` and the handoff test, retargeted to assert on the friction directory (no note, or one note) in place of the filed titles; card 4 extends it for the `loom-crash-resume.md` fields and drops the duplicate step-level and "voucher matches" and consume-once rows from its new tests.
The new table keeps only what no test covers: run lock held, state not `running`, empty history, Tier 2 off and the unwritable friction directory, as note-writer cases.

### [LOW] Card 5 does not pin every stated bound

**Location:** `_lyx/plan/05-start-vouches.md`, Intent and Tests.
**Issue:** The Intent says the live-strand no-op writes no voucher and that a write replaces any earlier voucher; the tests cover the unfinished-merge refusal, the parked-driver resume and Tier 2 off, not those two.
The one-shot and mismatch bounds are pinned by the renamed `TestConsumeHandoffVoucher` and `TestObserveEntry_...` tests, which the card does not say.
**Fix:** add a no-voucher assertion to the existing `TestRunDriverSpawnAndWait_LiveDriverOverRunningRunIsANoOp`, a replace row for a pre-existing voucher in the success-path test, and one sentence saying the consume side is pinned by the renamed existing tests.

### [LOW] Card 3's grep misses the prose it leaves stale

**Location:** `_lyx/plan/03-rename-handoff-voucher.md`, Intent last two paragraphs.
**Issue:** The closing grep matches `StepHandoff`, `stepHandoff`, `step-handoff` and `writeSelfreportStatus`, so it cannot see "step clean-handoff marker" and "clean-handoff marker" in the comments of `arm.go` (`loomPreStep`, `loomPostStep`, `observeStepEntry`'s neighbours), `halt_test.go` and `smoke_bootstrapwiring_test.go`.
**Fix:** extend the Intent to reword every comment in the Edit targets that says "clean-handoff marker", and add `clean-handoff` to the closing grep.

## Not defects (settled against the focus directive)

- Card 1's whole-file delete of `loomstatus_test.go` is right: the file holds only the four `LoomSelfreportFiled` and `LoomSelfreportFiledLock` tests.
- `awaiting_test.go` keeps `TestCheckCoherence_AwaitingStateAndHistoryPass`, and the `selfreport_test.go` helpers other files use (`writeSelfreportStatus`, `productJSON`) are named; `crashResumeEntry` and `haltStatus` are used only by deleted tests.
- Verify coverage: `go test ./...` runs the untagged files of every targeted package, the integration line names each of them, and `go vet -tags smoke` compiles the smoke files cards 2 and 4 edit (`smoke_test.go`, `smoke_bootstrapwiring_test.go`, `smoke_live_test.go`).
- Card 1 leaving `DetectCrashResume` and `ClassifyHalt` without a production caller until card 4 builds and tests cleanly once F1 is fixed; the repeated touches of `arm.go` are separate units (removal, key, rename, behavior).
- The `shedverbs/spec.go` comment edit follows from the rename and needs no separate licence; `RecentHistory` and `Vouched` are licensed by D3 and D4.
- Every decision and constraint is carried: D1 and D2 by cards 1 and 2, D3 by card 4, D4 by cards 3 and 5, D5 and D6 by card 6.
