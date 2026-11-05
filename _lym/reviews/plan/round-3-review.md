---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lym/plan/12-drift-expect-producer.md:72
    summary: "Card 12 prescribes a Drift-Expect Stuck for a record with no awaited_at, naming lym drift defer, and no TestDriftExpect row reaches it"
    fix: >-
      In _lym/plan/12-drift-expect-producer.md, in the TestDriftExpect coverage paragraph, add one row after the
      missing-baseline-directory row: a scope record with no awaited_at at Drift-Expect is a Stuck saying the capture never
      counted and naming lym drift defer, spawns no agent and writes no seal commit.
      Change the sentence that says each of those two Stucks' reasons opens with DecisionHaltPrefix followed by DeferHaltPrefix
      so it covers all three Stucks (moved warp head, missing baseline directory, capture never counted).
      No target changes, so no overlay is needed.
  - id: F2
    severity: BLOCKING
    class: scope
    location: _lym/plan/16-drift-verify-lines.md:104
    summary: "Card 16 prescribes an interim Stuck for a recorded command diff mode, naming lym drift defer, and no TestDriftVerify row reaches it"
    fix: >-
      In _lym/plan/16-drift-verify-lines.md, in the TestDriftVerify coverage paragraph, add a row: a scope record whose recorded
      diff is command is a Stuck naming lym drift defer, its reason opening with DecisionHaltPrefix then DeferHaltPrefix,
      with no judge spawned and no compare run.
      In _lym/plan/17-drift-verify-command.md, in the TestDriftVerify paragraph, say that card 17 replaces card 16's interim
      command-mode row with the command-mode rows it lists.
      No target changes, so no overlay is needed.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lym/plan/16-drift-verify-lines.md:107
    summary: "Card 16 derives the fix route from the rows Darn-Write and Webster, but the row constants for them are created by the later card 19"
    fix: >-
      Move the two Create lines for darnWriteRow and websterRow out of _lym/plan/19-drift-amend-defer.md and into a new
      **Create:** group on _lym/plan/16-drift-verify-lines.md (insert it before its **Extends:** group):
      the line - `plan:internal/driftengine#darnWriteRow` -> `const darnWriteRow = "Darn-Write"` and
      the line - `plan:internal/driftengine#websterRow` -> `const websterRow = "Webster"`.
      In card 19 keep the Create line for recheckRow, add `plan:internal/driftengine#darnWriteRow` and
      `plan:internal/driftengine#websterRow` under **Uses:** (after the expectRow line), and change the Intent sentence
      "this card's `recheckRow`, `darnWriteRow` and `websterRow`" to "this card's `recheckRow` and card 16's `darnWriteRow` and `websterRow`".
      In card 16's Intent, say the route derivation compares Deps.Rows against those two constants.
      Checked with lym loom validate-plan --overlay: the baseline run with {} and the run with this fix applied both report ok true
      and the same three informational findings, so the fix adds no blocking finding.
  - id: F4
    severity: LOW
    class: design
    location: _lym/plan/15-drift-judge-seat.md:68
    summary: "Card 15 exempts loom's run, friction and archive paths from the judge snapshot guard without stating what the judge can then change unseen or what bounds it"
    fix: >-
      In _lym/plan/15-drift-judge-seat.md, after the paragraph that lists the exempt paths, add a Bound sentence: an edit the judge
      session makes under the exempt run directory, friction directory or friction archive siblings goes unseen by the guard;
      those paths are loom's own run bookkeeping and hold no code, no expectation and neither scope.json nor judged.json;
      name what reads them afterwards and why a judge edit there decides no containment.
      No target changes, so no overlay is needed.
  - id: F5
    severity: LOW
    class: scope
    location: _lym/plan/05-drift-expectation-parser.md:42
    summary: "Cards 5 and 6 prescribe parser refusals that their named test rows never list"
    fix: >-
      In _lym/plan/05-drift-expectation-parser.md, add rows to the TestParseExpectation list for an n below 1 and for a case-level
      why under lines, both prescribed in the Intent.
      In _lym/plan/06-drift-judgement-parser.md, add rows to the TestParseJudgement list for a ruling outside the mode's set,
      an empty reason, a repeated case under command, a cells list under command, a cases list under lines and an unknown field.
      They are rows of the existing tests, not new test functions.
      No target changes, so no overlay is needed.
---

### [BLOCKING] Card 12's never-counted Stuck has no test

**Location:** `_lym/plan/12-drift-expect-producer.md:72` and `:81`
**Issue:** The `DriftExpect.Call` description says a record with no `awaited_at` is a Stuck, "the same way" as the missing-baseline Stuck: it opens with `DecisionHaltPrefix` then `DeferHaltPrefix`, says the capture never counted and names `lym drift defer`.
The input is reachable, since D12 names a run that passed Drift-Baseline before Drift-Await joined the recipe and then arrives at Drift-Expect.
`TestDriftExpect` (line 81) lists the moved-head Stuck and the missing-directory Stuck, with `awaited_at` set, and the sentence on prefixes covers "those two Stucks".
No row sets a scope record with no `awaited_at`, so nothing reaches this refusal.
`PATTERN-refusal-way-forward` requires a test to reach every new shed refusal, and the overview's constraints make that a blocking violation.

### [BLOCKING] Card 16's interim command-mode Stuck has no test

**Location:** `_lym/plan/16-drift-verify-lines.md:104`
**Issue:** The Intent says "until then a recorded `command` mode is a Stuck naming `lym drift defer`", which is a new shed refusal.
The `TestDriftVerify` paragraph (line 137) has no row for a recorded `command` mode, and neither does card 17's list, which replaces the refusal.
`PATTERN-refusal-way-forward` requires a test to reach it while the refusal exists.
The fix adds one row and lets card 17 replace it, so no test outlives the refusal.

### [MEDIUM] Card 16 needs row constants that card 19 creates

**Location:** `_lym/plan/16-drift-verify-lines.md:107`
**Issue:** Card 16 and the Shared Decision `tapestry-fix-route` derive the route from `Deps.Rows`: `Darn-Write` when the rows hold it, `Webster` otherwise.
That comparison is code in driftengine, and the only row-name constants the plan declares for it are `darnWriteRow` and `websterRow`, created by card 19, three cards later.
Today the package has only `verifyRow` (`internal/driftengine/verbs.go:21`), `captureRowBaseline` and `captureRowAwait`.
Card 16 either writes the two names as literals, which `PATTERN-named-constants` forbids once card 19 declares them, or depends on a later card, which breaks the plan-versus-schedule rule.
`expectRow` is correct: one constant from card 8, referenced by cards 10 and 19.

### [LOW] Card 15's exempt set has no stated bound

**Location:** `_lym/plan/15-drift-judge-seat.md:68`
**Issue:** The snapshot guard skips `judgement.yaml`, the run's shed directory, the friction directory and its archive siblings.
A judge session can therefore write there unseen.
The card states the exemption and its reason, "loom's own commits are never read as the judge's edits", but not what the exemption lets through or what bounds it, which the attack-surface rule requires of a guard with a hole.
The decision record's D9 bound lists only ignored paths and files outside both checkouts, so this hole is the plan's own.

### [LOW] Parser refusals without test rows

**Location:** `_lym/plan/05-drift-expectation-parser.md:42`
**Issue:** Card 5's Intent refuses an `n` below 1 and a case-level `why` under `lines`, and its `TestParseExpectation` list names neither.
Card 6's Intent refuses a ruling outside the mode's set, an empty reason, a repeated case, a `cells` list under `command`, a `cases` list under `lines` and an unknown field, and its `TestParseJudgement` list names none of them.
Both files are agent-written and cross a trust boundary, so each refusal needs a row in the existing table.

## Rejected

- Focus 1, removed or renamed `driftengine` Config, Paths and Deps fields (cards 4, 24, 25, 26): every test member that reads or sets a removed name is listed under the card that removes it.
  Checked by grep over every `_test.go` file: `config_test.go` `TestLoadConfig` and `templateConfig`, `seats_test.go` `TestSeatTables`, `baseline_test.go` `newBaselineFixture`, `seedScopeRecord` and `TestDriftBaseline`, `verify_test.go` `newVerifyFixture`, `compare_test.go` `TestRunCompare`, `capture_test.go` `TestRunCapture`, `entries_drift_test.go` `driftTestEnv`, `battencli`'s `intentionallyNil`, `driftcli`'s `driftYAML` and `TestCapture_OutlivesCaller` (which reads `paths.Runs` and is edited by card 22 before card 25 drops it), `configcli`'s `TestDispatchHubWideBoard`, and `loomcli`'s deleted `TestDriftDeps_AcceptedSeamsKeepAcceptedTestsInTheRun`.
- Focus 2, `TestWire_DarnRecipe`: card 24 lists it under Edit; cards 25 and 26 list it under Uses after card 24, and no earlier card lists it, so the later-target rule is not broken.
- Focus 3, card 9's lock wait: `CaptureRequest` gains `Sleep`, and the `TestCaptureCurrent` row injects it and asserts the waits of one second and then two, as `awaitCaptureEnd` takes its sleep.
- Focus 4, the `Drift-Expect` row name: `expectRow` is one constant declared once in card 8 and referenced by cards 10 and 19.
  `loomshed`'s `NameDriftExpect` is another package's constant, as `NameDriftBaseline` is today.
  The one literal problem found is the Darn-Write and Webster pair, filed as F3.
- Focus 5, the snapshot exempt set against `statusCommitPathspec`: the exempt run directory covers `StatusRel` and `SettleRecordsRel`, which `shedrun` joins under `RunDirRel`; the friction archive prefix matches `LoomFrictionArchivesRelPattern`, which `preflight.runStateExemptRels` also exempts; and the reviews and loom durable directories are judged by content, so a loom commit of content already uncommitted at the snapshot is no change.
  The exemption's missing bound is filed as F4.
- Focus 6, `judgement.yaml`: card 16 removes it with the rulings on a Done and on a does-not-match Stuck, and asserts it in `TestDriftVerify`.
  `fabricengine.Clean` is called only from `preflight` at run start.
  Card 16's seam list omits `Commit`, so Drift-Verify has no exit commit of `_lym/drift`; a file left by another Stuck stays untracked, which the decision record accepts because every spawn removes a stale one, though the driver's hand-over check would read it as dirt (`dirtyPathsOutsideRecords` exempts only the run, loom durable and reviews directories).
- Focus 7, `docs/overview.md`: card 22 names the verb list at line 279, the drift paragraph (lines 616 to 618) and the darn recipe sentence at line 534.
  The tail sentences at lines 614 and 615 stay true, since Drift-Expect sits before the writer.
- The `caller-uncovered` informational finding on `recordingSeats.Run` that `validate-plan --overlay {}` prints is the gate's own output, so it is not raised.
