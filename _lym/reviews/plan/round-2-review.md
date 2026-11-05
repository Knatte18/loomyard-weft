---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/plan/24-drift-retire-old-seats.md
    summary: Cards that drop a Config, Paths or Deps member do not list every test member that reads it, so the gates of cards 24, 25 and 26 do not compile
    fix: >-
      Edit these lists, then check them with lym loom validate-plan --overlay (the combined run below reported no new blocking finding).
      Card 24, add under Edit `internal/driftengine#TestLoadConfig`, `internal/driftengine#baselineFixture.seedScopeRecord`, `internal/driftengine#TestDriftBaseline`, `internal/driftengine#TestDriftAwait`, `internal/driftengine#TestDriftVerify` and `internal/loomcli#TestWire_DarnRecipe`, and remove `internal/loomcli#TestWire_DarnRecipe` from card 24's Uses (the same line is already in its own target list otherwise, which card-field-overlap refuses).
      Add to card 24's Intent that the subtest "the durable paths lie under DirRel" in `TestLoadConfig` drops `BaselineReport` and `VerifyReport` from its durable map, that `seedScopeRecord` drops its `report` parameter and its write of baseline.md and its callers drop the argument, and that the sub-run of `TestWire_DarnRecipe` which reads `c.env.Drift.BaselineTable("")` reads `c.env.Drift.ExpectTable("")` instead and keeps asserting that `subject_reading` holds the decision record path and no `lym board get`.
      Remove the line `internal/loomcli#TestWire_DarnRecipe` from the Uses list of cards 7, 8, 9, 12, 15 and 16, since the seam-test-unlisted check accepts a later card that edits the test (and uses-later-target refuses the Uses once card 24 targets it).
      Card 25, add under Edit `internal/driftengine#TestLoadConfig`, and say in its Intent that the same subtest drops `SideDir` and `Runs` from its ephemeral map.
      Card 26, add under Edit `internal/driftengine#baselineFixture` and `internal/driftengine#newBaselineFixture`, and say in its Intent that the fixture stops setting `Config.Skip` (field `configSkip` and the `TestDriftBaseline` row that sets it go).
      Method for the fixer: before a card that deletes or renames a field lists its members, grep every `_test.go` file for each removed name and list every enclosing member.
  - id: F2
    severity: BLOCKING
    class: design
    location: _lym/plan/09-drift-current-capture.md
    summary: The current capture's lock wait polls on package constants with no injected sleep, so its test waits out a one-second interval
    fix: >-
      In card 9's Intent, give `CaptureRequest` (already under Extends) a field `Sleep func(context.Context, time.Duration) error`, where nil sleeps for real, and have `CaptureCurrent` wait through it from the one-second floor, doubling to thirty seconds, as `awaitCaptureEnd` already takes its sleep as a parameter.
      Say that `loomcli`'s `driftDeps` leaves it nil.
      Change the `TestCaptureCurrent` row "finding the lock held waits and finishes once it is released" to pass a `Sleep` that releases the held lock and returns at once, and have it also assert the sequence of waits it was asked for (one second, then two) so the cadence the card states is pinned without waiting.
  - id: F3
    severity: BLOCKING
    class: consistency
    location: _lym/plan/08-drift-seal-baseline-checks.md
    summary: The row name Drift-Expect is a repeated literal in the prescribed PastRow calls of cards 8 and 10, with no named constant
    fix: >-
      Under card 8's Create add `plan:internal/driftengine#expectRow` -> `const expectRow = "Drift-Expect"` (the combined overlay check passes with it).
      In card 8's `CheckSeal` bullet and in card 10's `Check` step 3 write `PastRow(d.Rows, status row, expectRow)` in place of the literal, and say in card 19's `Amend` and `Defer` admission text that the rows they admit (the writer rows, Drift-Verify, Drift-Recheck, Drift-Expect) are named through constants declared once in driftengine beside `expectRow` and `verifyRow`, never as repeated literals.
  - id: F4
    severity: MEDIUM
    class: design
    location: _lym/plan/15-drift-judge-seat.md
    summary: The judge snapshot exempts fewer weft paths than loom's own commits stage, so a loom commit alone can make a resumed Drift-Verify Stuck again
    fix: >-
      In card 15's Intent change `JudgeSnapshot.Changes` to judge a weft path by content: a path that `WeftChangedSince`, `Changed` or `Untracked` names is no change when it was already uncommitted at the snapshot and its content now equals the hash or copy the snapshot recorded for it, so a later loom commit of content that was already there is not read as the judge's edit.
      Also add the friction archive siblings (a directory under `loomengine.LoomDurableDirRel()` for which `loomengine.IsLoomFrictionArchive` is true) to the exempt paths `loomcli`'s `driftDeps` passes beside `LoomWeftDirs`, as `preflight.runStateExemptRels` lists them.
      Add a `TestJudgeSnapshot` row: a file under the reviews directory that was untracked at the snapshot and is committed afterwards with the same content is no change, while a changed content or a new path there is named.
      No target list changes.
  - id: F5
    severity: LOW
    class: decision
    location: _lym/plan/16-drift-verify-lines.md
    summary: Nothing commits or removes judgement.yaml after the judge returns, so it stays untracked in the tracked _lym/drift directory
    fix: >-
      In card 16 step 10 say that once the changes check passes, Drift-Verify stores the rulings in judged.json and removes `_lym/drift/judgement.yaml` in the same step, on a Done and on a does-not-match Stuck alike, since the stored tuples are the record and every spawn removes any stale file anyway.
      Reword card 15's description of the exempt `judgement.yaml` as a file the judge writes, not one loom writes.
      Add a `TestDriftVerify` assertion to the Done row and the judge does-not-match row that `_lym/drift/judgement.yaml` is gone afterwards.
      No target list changes.
  - id: F6
    severity: LOW
    class: consistency
    location: _lym/plan/22-driftcli-retire-verbs.md
    summary: Card 22 edits docs/overview.md but names only its drift paragraph, leaving the darn recipe sentence that lists the rows in order without Drift-Expect
    fix: >-
      In card 22's Intent add that the sentence of docs/overview.md beginning "A second embedded recipe, `contracts/recipes/darn-recipe.yaml`" names Drift-Expect between Drift-Await and Darn-Write and the `drift` gate entry on Darn-Write, beside the drift paragraph it already names.
      No target list changes.
---

### [BLOCKING] Cards 24, 25 and 26 leave test members out of their Edit lists

**Location:** `_lym/plan/24-drift-retire-old-seats.md`, `_lym/plan/25-driftengine-retire-acceptance.md`, `_lym/plan/26-drift-retire-skip-keys.md`

**Issue:** The focus asked whether every test member that reads a dropped or renamed member is listed on the card that drops it.
Cards 4 and 16 pass: every reader of the renamed model keys (`config_test.go`, `seats_test.go`, `verify_test.go`, `configreg_test.go`) is on card 4, and every test that builds a DriftVerify row (`minimalEntries`, `engineMinimalConfig`, `driftTestEnv`, the motif fixture) is on card 16.
The later removal cards miss readers, each found by grepping the tree for the removed name:
- Card 24 drops `Paths.BaselineReport` and `Paths.VerifyReport`, but the subtest "the durable paths lie under DirRel" inside `TestLoadConfig` (`internal/driftengine/config_test.go`) reads both, and `baselineFixture.seedScopeRecord` (`baseline_test.go`) writes `f.paths.BaselineReport`. Neither is on card 24.
- Card 24 drops `Deps.BaselineTable`, but `TestWire_DarnRecipe` (`internal/loomcli/wiring_darn_test.go`, sub-run "a darn seed fills the darn seams...") calls `c.env.Drift.BaselineTable("")` and pins that the darn Drift agents read the decision record and never the board. Card 24 lists that test only under Uses, which is no edit, so the loomcli test package does not compile and the pinned behaviour is lost for the new seats.
- Card 25 drops `Paths.SideDir` and `Paths.Runs`, which the same `TestLoadConfig` subtest reads, and card 25 does not list `TestLoadConfig`.
- Card 26 drops `Config.Skip`, but `newBaselineFixture` builds `Config{... Skip: f.configSkip}` (`baseline_test.go`). Card 26 lists `TestDriftBaseline` but not the fixture or its struct.

The same enumeration method misses in three cards, so the finding is filed once as a method gap.
The overlay check in the fix passed with every proposed list change applied together; the Uses removals on cards 7, 8, 9, 12, 15 and 16 are required because uses-later-target refuses a Uses of a test a later card edits.

### [BLOCKING] The current capture's lock wait cannot be tested without waiting out its interval

**Location:** `_lym/plan/09-drift-current-capture.md`

**Issue:** The cadence question of the focus is settled: card 9 states a one-second floor doubling to thirty seconds under `PATTERN-daemon-wakeups`, and its Edit list names `pattern/PATTERN-daemon-wakeups.md`, whose background list is where the entry names the wait.
But `CaptureRequest` gains only `InheritedSlot`, and the interval is package constants, while `TestCaptureCurrent` has a row where a capture finds the lock held and waits until it is released.
`pattern/PATTERN-test-speed.md` ("Time is injectable") requires a production interval a test would otherwise wait out to be a struct field or parameter, and `PATTERN-daemon-wakeups` says a test never waits out an interval.
`wait.go`'s `awaitCaptureEnd` already takes its sleep as a parameter, so the precedent exists.

### [BLOCKING] Repeated literal row name with no named constant

**Location:** `_lym/plan/08-drift-seal-baseline-checks.md`

**Issue:** `PATTERN-named-constants` requires a name that can go stale to be one named constant referenced everywhere.
Card 8's `CheckSeal` and card 10's `Check` step 3 both prescribe `PastRow(d.Rows, status row, "Drift-Expect")` as a literal, and card 19's `Amend` and `Defer` admit rows by name with no constant either.
The package has constants for the existing rows (`verifyRow`, `captureRowBaseline`, `captureRowAwait`), and no card declares one for the new row.

### [MEDIUM] Judge snapshot exempts fewer weft paths than loom's commits stage

**Location:** `_lym/plan/15-drift-judge-seat.md`

**Issue:** The focus asked whether a resumed Drift-Verify row, with an open judge mark or after a judge-change Stuck, can be Stuck again by loom's own commits alone.
For the commits the card names, no: `status.json`, the driver records and settle records sit under the run's shed directory, and halt and crash-resume notes sit in the friction directory, both exempt, and the test rows cover them.
But `statusCommitPathspec` (`internal/loomcli/wiring.go`) stages, at every transition, the whole reviews directory and the whole `_lym/loom` durable directory as well, and a commit the pre-push check stops, or one skipped while a merge is active, leaves content there uncommitted until a later transition commits it.
Such content is untracked or changed at the snapshot and committed afterwards, so `WeftChangedSince` names a non-exempt path and the row is Stuck with no judge edit.
`preflight.runStateExemptRels` also exempts the friction archive siblings, which the card's exempt set omits.
The fix judges by content instead of widening the exemption to the whole reviews directory, which would let a judge write there unseen.
The new fabric seams of cards 1 and 2 are general and read-only: they name no drift concept, read through `gitrepo`'s in-process reads and `trackedDirtyPaths`, and `CommitAnchoredPathsWithTrailers` reuses the existing weft commit path, so `PATTERN-fabric-isolation` and `PATTERN-fabric-git` admit them.

### [LOW] judgement.yaml is never committed or removed after the judge returns

**Location:** `_lym/plan/16-drift-verify-lines.md`

**Issue:** Today every drift row commits `_lym/drift` on exit through `commitOnExit`; card 16's `driftVerifyEntry` no longer requires the `Commit` seam, and the records commits stage only `scope.json` and `judged.json`.
D9 puts `judgement.yaml` in the tracked `_lym/drift` directory, so after a pass it stays untracked in the weft.
`fabricengine.Clean` counts untracked weft files outside the exempt run state as dirt, and the driver's hand-over check reads paths outside the run's records directories, which do not include `_lym/drift`.
Card 15 also calls the file one "loom itself writes", which it is not.

### [LOW] docs/overview.md darn row order not named in card 22

**Location:** `_lym/plan/22-driftcli-retire-verbs.md`

**Issue:** The focus asked that `internal/loomrecipe/doc.go` and `docs/overview.md` are edited wherever they state the drift row order, route or verbs.
`internal/loomrecipe/doc.go` is covered (card 13 for the row orders, card 16 for the route), and card 22 covers the verbs line and the drift paragraph of the overview.
The overview's darn recipe sentence lists `Drift-Await` then `Darn-Write` with no Drift-Expect, and card 22 names only the drift paragraph.

## Rejected

- Judge snapshot against loom's status, driver-record and friction commits: covered by the exempt set and the `TestJudgeSnapshot` and `TestDriftVerify` rows; the residual gap is F4.
- Delete lists of cards 16 and 25: no member is in the Delete list of two cards (checked over every card's Delete list); every symbol card 25 deletes has its last caller removed earlier (`SkipList` callers go in cards 17 and 22, `CountedCapture` callers in cards 16 and 25, `ResetRuns` and `TakeRun` callers in cards 22 and 25), and `CountedCapture` and `TestCountedCapture` are both deleted.
- Refusals of cards 4, 9, 14, 19 and 26: each names a way forward that fits every state reaching it, and a named test reaches it (`TestPrepareRunSettings`, `TestLoadConfig`, `TestAmend`, `TestCheckParentCaller`, `TestAmend_RefusesWhatItCannotRead`, `TestDeferWaive`, `TestRunCapture`); `lym config drift --set` takes list values, so the `key_patterns` way forward holds.
- Drift-Baseline, Drift-Expect and `lym drift check` ordering: a row before Drift-Expect skips before any baseline check, a past row with no `awaited_at` reaches the never-counted fault, and no Drift-Baseline commit reaches `Unguarded` with a missing directory still faulting; the card tests seed states a real or legacy run holds.
- Recipe headers and comments naming a removed verb: `lym drift waive` in the darn header is replaced by card 18, and the Drift-Recheck row comments naming the accepted profile are rewritten by card 16.
- The informational caller-uncovered finding on `recordingSeats.Run` is a name after a dot in an unrelated package, as the gate reports it at the baseline too.
