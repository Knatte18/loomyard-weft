---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: internal/loomcli/integration_driverbootstrap_test.go:58
    summary: Card 03's idle wait before every verified send breaks the two loomcli stub providers, which print only ReadyFooterFixture and never an empty input box, so tmux-tier loomcli tests fail deterministically
---

Review range: no `_lyx/loom/rework/` exists, so the range is `git diff $(git merge-base main HEAD)..HEAD` (`product.parent` is `main`).
The review was formed from the diff, the plan under `_lyx/plan/` and the code; no prior-round file was read.
The focus directive for this round sets no focus (`focus: []`), so no departure is recorded.

Beyond F1, the diff matches the cards I checked: the send path in `internal/shuttleengine/send.go`, the stop verb in `stop.go`, the `PATTERN-shuttle-stop` entry in `PATTERN.md`, and the strand-removal call sites outside `internal/shuttleengine` (only `orchcli`'s own orch strands remain, which the pattern does not cover).
I raise no other finding.

### [BLOCKING] Stub providers in loomcli never draw an idle input box

**Location:** `internal/loomcli/integration_driverbootstrap_test.go:58-80` (`integrationWriteStubDriverScript`) and `internal/loomcli/smoke_driverstrand_test.go:78-90` (`writeStubDriverScript`); the code that makes them fail is `awaitIdleSession` (`internal/shuttleengine/send.go`), reached through `sendVerified` from `loadSkillTurn` (`internal/shuttleengine/run.go`).

**Issue:** Card 03 (`send-waits-for-idle`) makes every verified send wait until `busyReading` finds the pane both `StartupReady` and `claudeengine.IdleSession`.
`IdleSession` returns false unless `inputBoxInterior` finds a caret line between two horizontal rules with a blank interior.
Both stub scripts print only `claudeengine.ReadyFooterFixture` (`? for shortcuts`), which satisfies `Startup` but draws no input box, so `IdleSession` is never true.
The startup skill-load send therefore fails with `ErrSessionBusy` ("the pane is not idle"), `abandonStartup` tears the run down, and the tests fail.

I reproduced it: `go test -tags tmux -count=1 -run TestIntegrationDriverBootstrap_ReturnsWithoutWaitingOnTheDriver ./internal/loomcli/` fails in 61s with `load skills [scribe:prose]: ... the pane is not idle`, the pane showing only the launch line, `? for shortcuts` and `/color blue`.
`.lyx/verify/verify.log` records the same failure plus `TestSmokeDriverStrand_ReentrantAcrossThreeBootstraps` and `TestSmokeStart_NeverAttachesAndAddsNoOperatorStrand` timing out at 30s, in `publish_verify` at every Publish of this run.
These are the only two stub providers that stand in for `claude` (grep for `ReadyFooterFixture` and `stub-claude` finds no other).

The plan's verify block does not list `-tags tmux`, but the repo's publish gate runs these tests, and "existing tests keep passing" is part of the goal: the diff changed the readiness contract of every verified send without updating the fixtures that depend on it.

**Fix:**
1. In `internal/shuttleengine/claudeengine`, add an exported idle-box fixture beside `ReadyFooterFixture` in `startup.go`: a horizontal-rule line, a caret line with an empty box, a rule line, then the `? for shortcuts` footer, in the shape `IdleSession` and `Startup` both accept.
   Pin it with one row in the existing `IdleSession` table test (`session_test.go`) and one in the `Startup` table test (`startup_test.go`), asserting idle and ready respectively, so a change to either classifier shows up there rather than in a live-substrate test; add no separate test.
2. In both stub scripts, print that fixture once at start (in place of the bare footer) and again after each line the script reads, so the session reads idle before the next send, including the pointer send after the skill-load turn.
3. Check no other stub prints only `ReadyFooterFixture` (the grep found none beyond these two).
4. Verify with `go test -tags tmux ./internal/loomcli/` and the touched `claudeengine` package only; the machine crashes under a concurrent full-suite run, so do not run the full suite.
