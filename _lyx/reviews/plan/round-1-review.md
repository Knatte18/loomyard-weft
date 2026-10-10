---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/01-gated-turn-end-holds.md
    summary: Card 1 drops Run.startedFresh but leaves out internal/shuttleengine/run_resume_test.go, which reads the field at lines 85 and 121, so the package's tests stop compiling on card 1
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/01-gated-turn-end-holds.md
    summary: Card 1 never names TestPollEventsTick_OutstandingBackgroundWork, whose two fresh-gated at-once rows expect a Done that the change removes and whose fresh and arrived columns and doc comment state the old rule
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/01-gated-turn-end-holds.md
    summary: Card 1 reworks TestWait_GatedFreshRunWithOutstandingShellEvaluatesGateAtOnce into a hold test whose behavior TestPollEventsTick_GatedWaitingWithOutputFilesIsNotAnArrival and the awaited-shell row of TestWait_RecordsEndedShells already pin
  - id: F4
    severity: MEDIUM
    class: decision
    location: _lyx/plan/04-verify-gate-way-forward.md
    summary: Card 4 has NewVerifyGate call verifyFixPromptFile, which returns an error, but NewVerifyGate has no error result and carries no re-sign arrow, so the card leaves that error path undecided
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/plan/10-status-policies-read-the-classifier.md
    summary: Cards 10, 11 and 12 each let a partitioned ambiguous answer through a guard, but their Intents leave unstated what bounds that, and only card 9 states it
---

### [BLOCKING] Card 1 misses a test file that reads a field it deletes

**Location:** `_lyx/plan/01-gated-turn-end-holds.md`

**Issue:** Card 1 drops `startedFresh` and `gatedArrival` from `Run` (`internal/shuttleengine/run.go:294-297`) and edits only `waiting_test.go` among the test files.
`internal/shuttleengine/run_resume_test.go` reads `run.startedFresh` in `TestRunnerStart_Resume_WarningReachesRunAndRunStarts` (line 85) and `TestRunnerStart_NoResumeSessionID_NeverCallsCheck` (line 121).
Once the field is gone, the `internal/shuttleengine` test package stops compiling, so card 1's own gate fails, and the file sits outside the card's write surface.

**Fix:** Add `internal/shuttleengine/run_resume_test.go` to card 1's `**Edit:**` list.
In the Intent, drop the two `startedFresh` assertions and keep each test's `ResumeWarning` and `CheckResume` assertions, which still pin live behavior.

### [MEDIUM] Card 1 leaves the tick-level shell table on the old rule

**Location:** `_lyx/plan/01-gated-turn-end-holds.md`

**Issue:** `TestPollEventsTick_OutstandingBackgroundWork` (`internal/shuttleengine/waiting_test.go:207`) has two rows that expect `wantDone`: "a fresh gated run with outputs and a transcript-reported shell finishes done before any gated arrival" and the payload-reported variant.
D1 removes that exit, so both rows fail after the change.
The table's `fresh` and `arrived` columns exist only to drive `countsAtOnce`.
The rows "a gated run waits once a gated arrival has reached the gate" and "an attached or resumed gated run waits" differ only in those columns.
The doc comment above the test states the rule being removed.
The card's test paragraph names `TestWait_GatedFreshRunWithOutstandingShellEvaluatesGateAtOnce` and `TestWait_RecordsEndedShells` only, so the implementer learns about this table only from a failing gate.
The instruction "drop every `run.startedFresh = ...` assignment in the file" fixes compilation but leaves the two rows asserting the old rule.

**Fix:** Have card 1's test paragraph rework `TestPollEventsTick_OutstandingBackgroundWork`.
Flip the two fresh-gated rows to expect waiting, and drop the `fresh` and `arrived` columns along with any rows they leave as duplicates.
Reword the doc comment and its `//testtiming:keep` line so a gated run with files present always waits.

### [MEDIUM] Card 1's reworked hold test repeats behavior existing tests pin

**Location:** `_lyx/plan/01-gated-turn-end-holds.md`

**Issue:** Card 1 turns `TestWait_GatedFreshRunWithOutstandingShellEvaluatesGateAtOnce` into a test of three facts:

- A gated run with its output file present waits through the turn end without evaluating the gate.
- A later `STOP` runs the gate once and finishes `OutcomeDone`.
- No shell is recorded as ended.

`TestPollEventsTick_GatedWaitingWithOutputFilesIsNotAnArrival` (`waiting_test.go:126`) already pins the first two facts: zero gate calls on the waiting turn end, then exactly one on the next `STOP`, ending `OutcomeDone`.
The row "an awaited shell never lands in EndedShells" of `TestWait_RecordsEndedShells` pins the full sequence end to end: gated, shell outstanding, `STOP` step, `OutcomeDone`, no ended shell.
The only difference is an unawaited shell instead of an awaited one, and after D1 the two behave alike.
Under `PATTERN-test-economy`, that difference is a table row, not a reworked standalone test.

**Fix:** Delete `TestWait_GatedFreshRunWithOutstandingShellEvaluatesGateAtOnce`.
Add an unawaited-shell row to `TestWait_RecordsEndedShells`, beside the awaited-shell row: gated, `transcriptShellTask`, a `STOP` step, `OutcomeDone`, no ended shells.
Optionally add a gate-call count column, which the planned deadline row needs anyway for "evaluated exactly once at finalize".

### [MEDIUM] Card 4 leaves NewVerifyGate's error path undecided

**Location:** `_lyx/plan/04-verify-gate-way-forward.md`

**Issue:** Card 4 creates `verifyFixPromptFile(geom Geometry) (string, error)`, the `filepath.Abs` computation `Run` makes inline today (`internal/websterengine/runlevel.go:678`).
The card says "`NewVerifyGate` calls it to hand `newVerifyGate` the path".
`NewVerifyGate` returns `(shuttleengine.Gate, *VerifyGateNotes)` with no error (`internal/websterengine/verifygate.go:125`), and the card lists it with no re-sign arrow.
The implementer must therefore pick an option the plan does not license:

- swallow the error;
- panic;
- re-sign the exported constructor, which breaks its callers in `internal/websterengine/verifygate_integration_test.go` and `internal/webstercli/verify_parity_integration_test.go`, files the card does not list.

**Fix:** Decide the shape in the card, using one of these:

- Pass the path `Run` already computes into `NewVerifyGate` as a new parameter. Mark that with the re-sign arrow and list both integration test files under `**Edit:**`.
- If `Geometry.PromptsDir` is a told absolute path, make `verifyFixPromptFile` return a plain `string` with no `filepath.Abs`.

### [LOW] Cards 10 to 12 leave the bound on the weakened guard unstated

**Location:** `_lyx/plan/10-status-policies-read-the-classifier.md`; also `_lyx/plan/11-done-check-details-carry-the-lookup.md` and `_lyx/plan/12-answer-symbols-read-the-classifier.md`

**Issue:** Card 9 states the classifier's bound: the free-tag cap, read and parse failures falling back to not partitioned, and no tag held true by the building toolchain.
Card 9 explicitly lets nothing through yet.
The guards weaken in later cards:

- Card 10 lets a partitioned `ambiguous` answer pass `statusFindings` for Edit, Delete, Rename endpoints and `Uses:`.
- Card 11 lands a partitioned Create or Rename-new done-check.
- Card 12 lets a partitioned member's candidates stand as declarations in the plan-gate passes.

Each of those Intents says what now passes, but none says what bounds it.
The rubric requires the card that introduces a weakened guard to state both.
D6's bound reads: only an answer whose candidates provably never coexist passes, the Delete direction keeps blocking, and a Rename old side never accepts several declarations.

**Fix:** In card 10's Intent, add one sentence stating the bound in D6's terms.
Point at it in cards 11 and 12, or restate it there.

## Verdicts on the focus directives

- **Fidelity:** each of D1 to D6 and each Constraint is carried.
  - D1 is carried by cards 1 and 2.
  - D2 is carried by cards 3 and 4.
  - D3 is carried by cards 5, 6 and 7.
  - D4 is carried by card 8.
  - D5 is carried by card 11.
  - D6 is carried by cards 9, 10, 12 and 13, plus the Create and Rename-new rule in card 11.
  - The test-budget constraint is carried by card 9, the only card adding a top-level test.
  - The doc-lifecycle constraint is carried by cards 1 to 7 and 10 to 13 for their packages.

  Card 10's `createFindings` re-sign is licensed: the writer notes make `candidateList` the one renderer carrying the constraint, and the verdict stays `create-already-exists`.
  No card adds scope the record does not license.
- **Card order:** not a defect.
  Every re-signed planglyph function's callers sit in the same card or are already given `worktreeRoot`:
  - `LaterDeleteReferences`, `planGatePass`, `coverageSubjects`, `callerCoverageFindings`, `DetectDrift`, `DoneChecks` and `resolvePass` already take `worktreeRoot`.
  - `CanonicalizeHandles` has callers only in `planglyph.go` and its two test files, which card 13 lists.
  - `candidateList`'s two production callers are both re-signed in card 10, and cards 11 and 13 use it afterward.
  - The exported callers outside planglyph (`internal/planindex`, `internal/websterengine/recordbatch.go`) see no signature change.
- **Re-sign arrows:** present on every function whose parameters or results change:
  - `statusFindings`, `createFindings`, `candidateList`, `doneCheckVerdicts`, `answerSymbols`, `buildEditRegions`, `coveredByTarget`, `renameDeclSource` and `CanonicalizeHandles`;
  - `recordedSessions`, `contractFileStatus`, `contractDeleteClause`, `newVerifyGate` and `renderVerifyGateFindings`.

  The arrowless Edit members keep their signatures, except as F4 records for `NewVerifyGate`.
- **Attack surface:**
  - Card 1 states its bound: the run deadline and the liveness check.
  - Card 7's exit code is a new outcome on an existing verb, not a weakened guard, and the card states its scope (`stuck` and `dead` only; `done` and running stay 0).
  - Card 9 states its bound.
  - F5 covers the cards where the guard weakens.
- **Test economy:** only card 1 plans a redundant test (F3).
  The other cards extend existing tests as rows or assertions, and every test they name exists.
  Card 9's new `buildenv_test.go` covers a new file's behavior no existing test covers.
- **Verify coverage:** not a defect.
  `go test ./...` and `go test -tags integration ./...` cover every targeted package, `go vet` compiles the `tmux` and `llm` tiers, and the order matches the plan template's stated order.
