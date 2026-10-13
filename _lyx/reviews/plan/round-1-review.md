---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/00-overview.md (Shared Decisions, The D11 audit table) and _lyx/plan/10-shuttle-read-guards.md
    summary: The D11 audit lists lines by a hand-picked set of loops rather than by every loop, so it misses Info-or-above lines inside loops it names as audited, including a Warn that repeats every gate re-check in Run.handleGatedBoundary
    fix: >-
      In 00-overview.md's D11 table, add a row for `shuttleengine` `Run.handleGatedBoundary`, line `gate: pending send failed, entry stays pending`, verdict guard, card 10.
      Add a row for `shuttleengine` `Run.checkLivenessTick`, line `dismiss trust prompt (non-fatal)`, verdict guard, card 10.
      Add a row for `shuttleengine` `Run.checkLivenessTick`, line `persist run started failed`, verdict keep (once, when the run is marked started).
      Add keep rows, each with its once-per-event reason, for the loops the table leaves out: `reedengine` boot loop in lifecycle.go (`boot attempt`, `boot poll has-session check failed`, `tmux server up but session did not materialize`, `zombie boot, reaping socket before retry`: once per bounded boot attempt); `seatengine` `noticeSender.deliver` (`advisor notice dropped`, `advisor notice never landed before the sender stopped`: once per line); `fabricengine` coalesce.go push loop (its two `commits left unpushed` Warns: once per push); and `landingshed` `PRAlign.wait` (no Info-or-above line in its poll loop).
      Change the paragraph above the table so it says the audit swept every loop and polled path, and drop the claim that a line outside the named loops needs no verdict.
      In card 10, add `internal/shuttleengine#TestGatePending_FailedSendWarnsAndStaysPending` under **Edit:**.
      Add to card 10's Intent: `handleGatedBoundary`'s pending-send failure, repeated on every gate re-check (`gateRecheckEvery`) while the send fails, becomes `run.failures.Fail("pending:"+entry name, …)` with its existing message and fields, and a pending send that succeeds calls the matching `Recover` with `shuttle: gate: pending send delivered again`.
      Also add to card 10's Intent: `checkLivenessTick`'s trust-prompt dismissal failure, repeated on every startup probe while no input is sent, becomes `run.failures.Fail("startup dismiss", …)`, and a dismissal that sends calls `Recover`.
      Add to card 10's Tests: `TestGatePending_FailedSendWarnsAndStaysPending` scripts the pending result on three evaluations before the pass, captures at Debug, and asserts one `level=WARN` and at least one `level=DEBUG` `pending send failed` line.
      Also add to card 10's Tests: a `TestRun_Wait_StartupWindow` row whose trust dismissal fails on every probe asserts exactly one `level=WARN` `dismiss trust prompt` line.
      Change card 10's opening sentence from four sites to the sites the table lists for the package.
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/03-reed-resize-episode.md, _lyx/plan/10-shuttle-read-guards.md, _lyx/plan/11-orch-probe-holds.md
    summary: Five once-per-episode guards prescribe an Info recovery branch that no named test reaches; their tests assert only the single Warn
    fix: >-
      Extend the existing test rows the cards already list, adding no new test function or target.
      Card 3: the `UnreadableClientListStillRenames` row of `TestRepairNames_SessionName` runs the repair a third time on the same engine with the client list readable, and asserts one `level=INFO` `operator activity readable again for session name repair` line.
      Card 10, activity guard: one `activityErr` row of `TestSend_WaitsForIdleSession` (or a new row in that table) has the fake's `OperatorActive` error on its first polls and then answer, and asserts one `level=INFO` `operator activity readable again` line beside the single WARN.
      Card 10, expiry guard: the probe-error row of `TestGatePending_ExpiryProbeEndsMidTurnWait` has the probe error on its first re-checks and then answer, and asserts one `level=INFO` `expiry probe answers again` line.
      Card 10, startup-capture guard: `TestRun_Wait_StartupWindow` gains a row whose pane capture fails on the first probes and then succeeds, and asserts one WARN and one `level=INFO` `startup probe capture works again` line.
      Card 11: the network-marker row of `TestWatcher_HubPauseInterruptsAtGraceEndHoldsAndQueuesTheResumeLine` replaces the plain file at `pauserecord.NetDir(e.pausePath)` with an empty directory after its failing ticks, ticks once more within the grace, and asserts one `level=INFO` `network operation markers are readable again` line.
      Each of these rows captures at Debug or Info as the assertion needs and stays serial.
---

### [MEDIUM] The D11 audit table misses lines inside loops it calls audited

**Location:** `_lyx/plan/00-overview.md`, Shared Decisions, "The D11 audit"; `_lyx/plan/10-shuttle-read-guards.md`

**Issue:**
D11 requires the plan to list "every Info-or-above line, Warn included, that sits in a loop or polled path, each with a verdict".
The overview instead names a fixed set of loops it covered, and declares any line outside them out of the audit.
That narrowing is not in the decision record, and it misses lines even inside the loops it does name.

The concrete miss that needs a code change is in `internal/shuttleengine/wait.go`, `Run.handleGatedBoundary`.
`shuttle: gate: pending send failed, entry stays pending` is logged at Warn on every gate re-check while the pending send fails, for example while the writer pane stays busy.
The re-check repeats every `gateRecheckEvery` (5 s).
This is the same function whose sibling `re-prompt not delivered` line card 10 demotes, but the table has no row for the pending-send line, and no card changes it.
`TestGatePending_FailedSendWarnsAndStaysPending` reaches the line today, but with one pending evaluation only.

Two more lines in `Run.checkLivenessTick` have no row, though the table and card 10 list that function as audited.
`dismiss trust prompt (non-fatal)` repeats on every startup probe while `playInputsSent` sends nothing, since `run.dismissed` is set only when `sent > 0`.
`persist run started failed` fires once.

Loops outside the named set also carry Info-or-above lines and get no verdict:
the reed tmux boot loop in `internal/reedengine/lifecycle.go`;
`seatengine`'s `noticeSender.deliver` retry loop;
the push loop in `internal/fabricengine/coalesce.go`;
and `landingshed`'s `PRAlign.wait` poll loop, which has no such line.
They read as keep: each fires once per bounded attempt or per event.
D11 still requires each one listed.

This is one gap in how the sites were found, filed once rather than per missed line.
The `fix` key's added test target was checked with `lyx loom validate-plan --overlay` against a `{}` baseline, and it adds no blocking finding.

### [MEDIUM] Recovery branches of several guards have no test that reaches them

**Location:** `_lyx/plan/03-reed-resize-episode.md`, `_lyx/plan/10-shuttle-read-guards.md`, `_lyx/plan/11-orch-probe-holds.md`

**Issue:**
Each of these guards prescribes three branches: Warn at the first failure, Debug while it repeats, and an Info line at recovery with its own message.
The plan template says a card prescribing a guard branch names the test that reaches it.
Card 2's `TestEpisode_WarnsOncePerEpisodeAndInfoOnRecovery` pins `Episode` in isolation.
It cannot catch a call site that never calls `Recover`, or calls it on the wrong seam's success.

Cards 8, 12 and card 11's tick-failure guard reach all three branches.
These five guards assert only one Warn, and no named test reaches their recovery line:

- card 3: `Engine.activityFailures` in `repairSessionName` (`operator activity readable again for session name repair`);
- card 10: `operatorActivityHold`'s per-strand set (`operator activity readable again`);
- card 10: `probeExpiry`'s set (`expiry probe answers again`);
- card 10: `checkLivenessTick`'s startup-capture episode (`startup probe capture works again`), whose two listed rows fail on every probe and so never recover;
- card 11: `netOpsFailures` in `orchNetworkOperationInFlight` (`network operation markers are readable again`).

The fix extends rows of tests the cards already list, so it adds no target and needs no overlay run.

## Rejected

- Card 4 bounds the new retirement edge as D2 and the focus ask.
  Its Intent states that a failed stat or a nil baseline counts as unchanged, that the check runs only on the timer tick and adds no wakeup, and that each process retires at most once.
  It also states that a multi-step deploy chains retirements until the file stops changing, and that a failed respawn waits for the next `reed up`, `attach` or `resume`.
  `runWatchdogLoop`'s three callers (`watchdogCmd`, `TestWatchdogDaemon` with two calls, `startReapLoop`) are all listed.
  Not a defect.
- Cards 9 and 11 are in the right order: card 11 uses `IdleProbe.Draft`, `IsDraftHold` and `ErrForeignBoxText` from card 9, and card 10 edits `busyReading` after card 9 re-signs it.
  Card 14's use of card 13's `LogArchiveDir` also comes after card 13.
  No card depends on a later one.
- Card 1's restore list matches every `logger.SetOutput(os.Stderr)` in test code (grep over `internal` and `cmd`).
  `tools/sandbox` is rightly excluded.
- Card 2's `PATTERN.md` arithmetic holds: the file is 15560 bytes now, the four shortened lines save 102 bytes, and the new entry line is 95.
- Card 5's per-value seen-set keeps state across ticks, because `cliwire.SessionNamer()` builds one `claudeengine.New()` per `WatchNames` call, not one per tick.
- Card 15's Warn for a notice that cannot be queued is `rowNotifier.send`'s existing hold-back.
  That path appends the notice log line before the check, as D16 requires.
- Card 16 covers D17's routing, its two next-wake board checks, the named-only fallback with its bound, and the deploy-recovery check scoped to an empty `deploy` key and `require_pr_to_base`, with claim rows for each.
  It adds no scope.
- Verify follows the template's order and runs both the untagged and `integration` tiers module-wide, compiling `tmux` and `llm` only.
- `TestExeReplaced` and `TestLoopReadFailuresLogOncePerEpisode` call unexported helpers, but each row pins a distinct behavior that no existing test covers.
  That is no `PATTERN-test-economy` violation.
