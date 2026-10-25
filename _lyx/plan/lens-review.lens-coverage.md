```yaml
lens: coverage
outcome: reported
```

Coverage lens: checking that every decision, acceptance criterion and branch in the plan has a test row that would fail if it broke.

- **BLOCKING — D4/D3 (loom detached capture), card 5 `StartCapture`/`RunCapture` with card 9 `DriftBaseline.Call`.** The detached child can refuse its own capture.
  - Card 5 `RunCapture` takes `CaptureLock`, then runs `Check`, which re-reads the status for `CheckMainCapture`.
  - `StartCapture` returns as soon as `ProbeLock(CaptureLock)` reads held. Card 9 then returns Done, and shed moves the current row to `Plan-Write`.
  - The child's `Check` can then see current row `Plan-Write` and refuse, since `CheckMainCapture` admits only `Drift-Baseline`, or `Drift-Await` while blocked. The capture writes no record (card 5: "`Check`'s refusal writes no record"), so `Drift-Await` halts on a missing record.
  - The same race hits `lyx drift capture` at a `Drift-Await` halt followed by a fast `lyx loom resume`, which sets the state to running.
  - No row covers the ordering. `TestStartCapture` only fakes "the fake child takes the lock".
  - Evidence: card 5 Intent ("runs `Check` … under the lock"; "returns the pid once `ProbeLock(CaptureLock)` reads held"); card 7 Intent (`capture-child` … "`Check` re-reading the status").
- **MEDIUM — acceptance criterion "flag turned off while `Drift-Await` waits or is halted, or while `Drift-Verify` is halted, skips neither row".** Card 14 argues this holds through shed's `EnteredRow` and plans no row. `TestDriftRows_EntryGate` checks only that the gate holds or does not hold. Evidence: card 14 Intent ("the gate is evaluated only when the run enters the row…"); `internal/shedengine/entry.go` `Shed.enter` (`st.EnteredRow == def.Name`).
- **MEDIUM — acceptance criterion "a capture started by the row outlives the row's step, on Windows too, and teardown ends it".**
  - Windows is deferred to the sandbox hub (Shared Decision "Windows job behavior proved outside this hub's gates"; card 4; card 13).
  - On Linux, the claim that `runproc.EndSparing` ends a `RoleCapture` child is stated without a row: card 13's rows only pin the Windows seam's call order.
  - `TestCapture_OutlivesCaller` covers the `lyx drift capture` caller, not the row's step child.
- **MEDIUM — D6/acceptance criterion "under scope `full` every `Drift-Verify` halt but an error or transient stop is a decision halt".** Card 11 says every Stuck under `full` opens with `DecisionHaltPrefix`. No row asserts the prefix on:
  - the dirty-entry Stuck after its attempts are spent (the row "left dirty, is Stuck" names no scope or prefix),
  - the verify-gate Stuck after its attempts are spent (no row exists for verify exhaustion at all),
  - the undeclared-compare Stuck of step 5.

  Step 8 says those Stucks "keep the gate's reason", so they need prefixing that no row checks.
- **MEDIUM — D1/acceptance criterion "with the flag on, an undeclared `capture` or `compare` is refused naming `lyx config drift`".**
  - Card 11 step 5 (`Config().Step("compare")` refusing) has no `TestDriftVerify` row.
  - The `lyx drift compare`/`capture` verbs' `Step` refusals have no row.
  - Card 7's `capture --side` path lists `CheckVerifyRow`, `TakeRun`, `RunSideCapture` but no `Config.Step("capture")`, so the side capture's step source and its undeclared refusal are unstated.
- **MEDIUM — acceptance criterion "`lyx drift compare` and `lyx drift capture --side` refuse … outside `Drift-Verify`".** `CheckVerifyRow` (card 7) appears in no planned test. `TestRecordScope`, `TestAcceptWithdraw` and `TestDeferWaive` do not name it. The verbs' order (row check, then `TakeRun`, then the run) has no row either.
- **MEDIUM — D6 resume detection, card 11 step 2.** Two branches have no row:
  - the `LastAt` branch, for folded identical Stucks;
  - the negative branch, where the last history entry is not a Stuck of this row or predates `halt_named_at`, so nothing is ruled.

  The only row uses a plain history Stuck at or after the recorded time.
- **MEDIUM — D3 "a capture holding the lock is … left running in loom", card 9.** No row covers the branch where the lock is held without `WaitCapture` and the row returns Done without starting a capture. The only lock row is under `WaitCapture`.
- **LOW — acceptance criterion "`partial` writes the skip file and passes it to capture and compare".** Card 6 has a skip-file row for the compare. Card 5's `TestRunCapture` rows never assert the capture's skip file or `{skip_file}` rendering.
- **LOW — acceptance criterion "on Windows with breakaway refused, loom's foreground capture is Stuck on a failure naming `lyx drift capture` and `lyx drift defer`".** Card 9's row asserts only "`ErrBreakawayRefused` falls back to the foreground run", not the Stuck reason after a failed fallback.
- **LOW — acceptance criterion "the scope record, reports and logs are committed with the run's records".** The rows only assert that `Commit` is called. Card 14's `driftDeps` `Commit` (`CommitAnchoredPaths` over `DirRel()`) has no row checking its pathspec.
- **LOW — D1 "executable keys, read only from the template and the board default layer".** Card 1 relies on `CategoryExecutable`. `TestLoadConfig` has no row where a local layer setting `capture` or `compare` is refused.
- **LOW — untested small branches.**
  - Card 1: the `LoadConfig` "not initialized" → `lyx fabric reconcile` mapping.
  - Card 5: `ProbeLock` on a missing directory reads not held and creates nothing.
  - Card 12: the probe-error return of `agentActivityFor`.
  - Card 7: the `lyx drift status` verb.
  - Card 8: no row tells the `decision-record` subject from the `board-entry` subject in the baseline table.
- **LOW — D5 "each read again after a lock check".** No `TestDriftAwait` row pins that the capture record is read after the lock probe, for example a record that flips to `passed` while the lock is released between probe and read.

No further tool calls made.
