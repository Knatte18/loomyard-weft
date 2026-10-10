---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: internal/shuttleengine/waitshadow.go:76
    summary: loopAgreesWithState still states a reason for the held-beside-busy-on-background allowance that the fold does not produce, and shuttleengine/doc.go still describes the retired shell expiry
  - id: F2
    severity: LOW
    class: consistency
    location: internal/shuttleengine/waiting_test.go:241
    summary: three files the diff touches are not gofmt-clean, one of them with two table rows joined on a single line
  - id: F3
    severity: LOW
    class: design
    location: internal/websterengine/begun_test.go:53
    summary: TestEditedDoneCards nil-state assertion duplicates the TestNewPlanGate no-run-record row, and the test's doc comment says the gate rows do not reach it
  - id: F4
    severity: LOW
    class: design
    location: internal/shuttleengine/waiting_test.go:63
    summary: the waiting-with-output-files-is-done row of TestPollEventsTick_Waiting duplicates rows of TestPollEventsTick_OutstandingBackgroundWork
---

Range reviewed: `status.json` gives `product.parent` = `main`, no `_lyx/loom/rework/` round exists, so the range is `git diff $(git merge-base main HEAD)..HEAD` (32 commits, 138 files).
`lyx gate test` over every touched package and `lyx gate test --tags integration` over the touched packages with an integration tier were green.

### [MEDIUM] loopAgreesWithState states a reason the code does not produce, and doc.go still describes the retired shell expiry

**Location:** `internal/shuttleengine/waitshadow.go:76`, `internal/shuttleengine/doc.go:51`, `internal/shuttleengine/doc.go:267`, `internal/shuttleengine/waitshadow_test.go:117`

**Issue:**
The doc comment says a held or done run may sit beside `busy`/`background` "since a turn end counted at once, or a plain Stop after a waiting one, leaves the session running tasks the loop no longer waits on".
The done half holds: a turn end counted at once leaves the fold at `busy/background` (`judgeTurnEnd` returns it for any signal with outstanding tasks), and the row `done beside busy on background work is expected` pins it.
The held half does not.
A plain Stop carries no outstanding list, so `judgeTurnEnd` reads it as `idle-stalled/no-output`, never `busy/background`.
The first row of `TestWait_LogsSessionStateBesideItsClassification` runs exactly that sequence (`WAIT`, then `STOP:what now?`) and expects `idle-stalled/no-output held` with no disagreement.
`heldTurnEnd` no longer carries tasks, and `Wait` clears `heldStart` on the same tick any new event is read, before `logSessionState` runs, so I found no path that pairs a held loop with `busy/background`.
The `|| busyOnBackground` arm of `case loopHeld` therefore has no producer and no row, and the comment's reason for it is contradicted by the repo's own test.

The row that used to pin the held pairing was renamed to `a waiting turn end on transcript-reported shells past the bound beside busy on background work is expected`.
Card 22 asked for a row about a held turn end beside busy background work.
The renamed row exercises a waiting loop, which already agrees with `busy` without the allowance, and its expected change `busy/background waiting` repeats the first row's.

`doc.go` still carries the old model in two places the card's rewrite did not reach.
Line 51 says a held turn end may be "an expired-shell turn end", but `expiredTurnEnd` no longer returns a held turn end and nothing expires.
Line 267 lists the expected pairing "a held turn end after transcript-reported shells expired beside busy on background work", which names the expiry the diff removes.

**Fix:**
Decide whether the held allowance is reachable.
If it is not, drop `|| busyOnBackground` from `loopHeld` and reword the comment and `doc.go:267` to the done pairing only; the "three pairings" count in `doc.go` and the comment becomes two.
If a real path exists, name it in the comment and add the row that exercises it.
Reword `doc.go:51` to "a Stop or a live ask".
Drop or repurpose the renamed `waitshadow_test.go` row so it does not repeat the first row's `busy/background waiting`.

### [LOW] Three touched files are not gofmt-clean

**Location:** `internal/shuttleengine/waiting_test.go:241`, `internal/planparser/validate_test.go:2134`, `internal/testkit/envkit/envkit.go:36`

**Issue:**
`gofmt -l` lists all three, and each was clean at the merge base.
`waiting_test.go:241` holds two table rows on one line, `{name: "an ungated run with outputs and nothing outstanding finishes done", …}` followed by a tab and the next row; the commit that added the row (`3d014e03e`, F3 of round 1) replaced the following row's line instead of inserting before it.
`validate_test.go:2134` and `:2135` read `entryLine(limit,"x")` and `entryLine(limit,"é")`.
The `"WebsterRecord"` key added to `nilLegal` in `envkit.go` changes the alignment `gofmt` wants for the whole map.

**Fix:**
Run `gofmt -w` on the three files.
Split the `waiting_test.go` rows onto their own lines.

### [LOW] TestEditedDoneCards repeats a case the plan gate's rows already cover

**Location:** `internal/websterengine/begun_test.go:53`

**Issue:**
The second assertion, `EditedDoneCards(plan, nil, planDir)` returning `nil, nil`, is reached by `TestNewPlanGate/NoRunRecordReportsNothing` in `internal/loomshed/gates_test.go`: with no saved record `LoadState` returns `(nil, nil)` and `ValidatePlan` passes that nil state straight to `EditedDoneCards`, where the gate asserts no `done-card-edited` finding and a passing gate.
The test's doc comment says it "pins what the plan gate's rows do not reach: … and a nil state reports nothing", which is false for the nil state.
The first assertion, a recorded id the plan lacks, is not reached by any gate row and stays.
The gate rows' names do match the fixtures they build.

**Fix:**
Drop the nil-state assertion and reword the doc comment to the recorded-id-the-plan-lacks case alone.

### [LOW] A TestPollEventsTick_Waiting row duplicates the new background-work table

**Location:** `internal/shuttleengine/waiting_test.go:63`

**Issue:**
The row `waiting with output files is done` runs an ungated run with output files present over a waiting turn end and expects `OutcomeDone` with no held turn end.
`TestPollEventsTick_OutstandingBackgroundWork` now holds `an ungated run with outputs and a transcript-reported shell finishes done`, `an ungated run with outputs and a fork finishes done` and `an ungated run with outputs and nothing outstanding finishes done`, and the rule it pins is the same whatever is outstanding.
Those rows additionally assert the tasks are counted and the waiting list cleared, so the older row adds no assertion of its own.

**Fix:**
Delete the `waiting with output files is done` row and the `touchOutput` field it alone uses; update the test's doc comment to the two remaining cases.

## Focus verdicts

Focus 1, `runlevel.go` step-keyed texts: not a defect.
`TestRun_AutoRebaseline` executes the quarry-unavailable transient (`quarry unable to answer`) and the plan-validation refusal after an accepted rebaseline; each asserts `ErrAutoRebaseline`, the re-step text, neither manual verb, and no `Start`.
The step-keyed zero-batch text is executed by `TestZeroBatchesErrorText` against the helper directly.
No plan reaches `recordedBatches`' zero-batch branch after an accepted rebaseline, because the rebaseline records a partition covering every card and the identity batcher gives at least one batch for any card, so the comment above the `Run` table and the helper-level test are the right pin; card 4 prescribes the `zeroBatchesError(planDir, step)` signature and the step pass-through.

Focus 2, `wait.go` ungated waiting turn end with nothing outstanding: not a defect.
`pollEventsTick` returns `OutcomeDone` for an `EventWaiting` with an empty list, no gate and every output file present, ahead of `expiredTurnEnd`'s empty-list guard.
The row `an ungated run with outputs and nothing outstanding finishes done` pins it, and `completionsignal_enforcement_test.go` records the second file-contract site in `pollEventsTick`.
The row is on the malformed line of F2 but runs.

Focus 3, `TestEditedDoneCards` against `TestNewPlanGate`: one duplicate assertion, F3; each gate row's name matches its fixture.

Focus 4, `loopAgreesWithState` comment: the stated reason does not describe what the fold produces for the held case, F1.
