---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: _lyx/plan/05-poll-mode-backoff.md
    summary: The card leaves the test PollCeiling unnumbered and bounds it only by the 300 ms five-tick tests, missing the 20 ms stability windows of TestWatchLoop_PollModeGoesDormantOnVanishedWorktreeRoot
  - id: F2
    severity: LOW
    class: design
    location: _lyx/plan/05-poll-mode-backoff.md
    summary: TestWatchLoop_PollModeByDefault's planned assertion, a list-panes count under what a fixed cadence would reach, also passes for a fixed-cadence loop, so it does not observe the backoff
---

### [LOW] The test ceiling's bound misses the dormancy test's 20 ms stability windows

**Location:** `_lyx/plan/05-poll-mode-backoff.md`, the Tests paragraph on `watchdogTestTiming`.

**Issue:** The card sets `PollCeiling` in `watchdogTestTiming` to "a small multiple of its `PollCycle`" and justifies it only by the 300 ms windows of `TestWatchLoop_OpenerFailureStaysInPollMode` and `TestWatchLoop_NonSentinelFailureDoesNotGoDormant`.
Any ceiling up to about 80 ms satisfies that bound (5 + 10 + 20 + 40 + 80 ms of waits fit inside 300 ms).
`TestWatchLoop_PollModeGoesDormantOnVanishedWorktreeRoot` (`internal/reedengine/watchloop_test.go:808`) also stays in poll mode on unchanged ticks: `newWatchLoopTestEngine` answers a constant `100 21` box, so every tick after the first doubles the wait.
After removing the worktree root it waits for `list-panes` to stop growing across one 20 ms window, then waits another 20 ms and asserts exactly one dormancy warning.
Dormancy is only detected by a poll tick's re-apply.
With a ceiling above 20 ms, a backed-off loop can look "stable" for a 20 ms window just because it is waiting.
The test then reads the log before the dormancy tick has run and fails on zero warnings.
The `eventually` that gates the removal usually returns after two ticks, so the wait is still short and the race seldom fires, but nothing in the card rules it out.
The card says loop tests whose ticks change something "keep passing as it is", but this test's ticks change nothing, and the card doesn't name it.
Every other poll-mode loop test was checked against a ceiling at or below 20 ms and passes: `PollModeByDefault`, `TakeEffectBoundary`, the two named 300 ms tests, and `UndecidedProbeDoesNotGuess`, whose deferrals return to the base.

**Fix:** Name the value in the card, for example `PollCeiling: 4 * PollCycle` (20 ms) or lower.
State the bound as both constraints: five backed-off ticks inside 300 ms, and a ceiling no longer than the 20 ms stability window `TestWatchLoop_PollModeGoesDormantOnVanishedWorktreeRoot` and `waitForPromotion` use.
Then list that test beside the two the card already names.
To settle the method, not just this one test: sweep `watchloop_test.go` for every loop test that stays in poll mode on ticks that don't return the wait to the base. Check each one's timing windows (the 300 ms `eventually` bounds and the 20 ms stability windows) against the ceiling, and list the result in the card.
Round 1 named one such test and its fix found a second; this third one shows the per-test approach keeps missing cases.

### [LOW] PollModeByDefault's new assertion cannot tell a backed-off loop from a fixed cadence

**Location:** `_lyx/plan/05-poll-mode-backoff.md`, the Tests paragraph on `TestWatchLoop_PollModeByDefault`.

**Issue:** The card says the test "asserts that the `list-panes` count over its window stays under the count a fixed cadence would reach, computed from the test timing's base and ceiling, so the backoff is observed on the loop itself".
A real fixed-cadence loop never quite reaches its ideal count over a wall-clock window, because scheduling and the fake's round trips add latency.
A regression back to a fixed ticker would therefore also stay under that count and pass.
The pure rule is pinned by the new `PollWakeChangedPerOutcome` step.
But the wiring that feeds `NextWakeCadence` and re-arms `wake` is what this assertion claims to observe, and as worded it observes nothing.

**Fix:** Make the bound the backed-off schedule's own count plus a fixed slack, and pick a window long enough that a fixed cadence's count clears that bound by a wide margin.
For example, with a 5 ms base and a 20 ms ceiling, a 200 ms window gives about 11 backed-off ticks against about 40 fixed ones, so an upper bound near 15 separates the two.
Alternatively, assert that the gap between two later consecutive `list-panes` calls is at least twice the base.
Name the chosen bound in the card.

## Focus verdicts

**Focus 1, the unresolvable-shell refusal on the shared fixture: not a defect.**
The refusal belongs to card 9, not card 7; card 7 adds only the socket-length check, and under `tmuxkit.Main`'s short `TMUX_TMPDIR` that check passes for every fixture.
The boot's pre-tmux block runs at the top of `ensureServerAndSessionLocked`, which `Up`, `Resume`, `upLocked` (and through it `EnsureSession`, `Revive` and `AddStrand`'s cold path) all reach.
I checked every untagged `reedengine` test that calls one of them:
- `TestUp_BootValidation` and `TestEnsureServerAndSession_RevivalTrigger` reach the block; both files are card 9 targets, and the card handles both (`reviveTestEngine` gets `sh`, and the `ValidWatchdog_*` rows keep their not-the-watchdog-error assertion).
- `TestRevive_CreatesNothingWithoutThePredicate` returns before `upLocked` in both its rows.
- The `AddStrand` and `AddStrandUnless` tests in `strand_test.go` and `spawn_test.go` stay on the warm path. The fake answers `has-session` with success by default, and `installIfAbsentTmux` or `launchFake` answer `list-panes`, so `ensureSessionLocked` never calls `upLocked`. `TestAddStrand_ColdEngine_NoLongerReturnsNoSessionMessage` installs no fake and fails at the substrate probe, before the boot.
- The other fixtures in `lock_test.go` that set a nonexistent shell never reach the boot.

Outside `reedengine`, no untagged test calls `Up` or `Resume` on a real engine.

**Focus 2, the probe server reading `~/.tmux.conf`: not a defect in the current plan.**
The Shared Decision `probe-server-bound` now gives the probe's `-V` and `list-commands` `-f /dev/null` through a `configFile` field on a copy of the engine's `TmuxCmd`.
That closes the case where a probe-started server reads the operator's config, keeps a session alive, and gets joined by reed's `new-session`.
The other pre-spawn calls (`has-session`, `list-sessions`) don't start a server.
`execHook` receives the argv after `serverArgs`, so `TestProbeCapabilityLocked_GoesThroughTheSocketScopedTmuxCmd` still holds.
Card 9's tmux-tier test seeds a config that creates its own session, which covers the probe path as well as the spawn.

**Focus 3, card 5's loop tests under the test ceiling:** the two named 300 ms tests pass under any ceiling the card's stated bound admits.
`TestWatchLoop_PollModeGoesDormantOnVanishedWorktreeRoot` is the unnamed test that needs a tighter bound (F1).
The card's stated reasons match: unchanged ticks double, failed ticks double, and the first tick applies because `lastApplied` starts as the zero box.
The doc follow-ups match the two named tests' comments.
`watchDefaultTimingMatchesTheSixConstants` gains the field as planned, and `cmd/lyx/daemonwakeups_test.go` already lists `internal/reedengine/watchloop.go`.

## Focus departures

Focus 1 reads: "Card 7 makes `ensureServerAndSessionLocked` refuse an unresolvable shell in its pre-tmux block".
The shell refusal is card 9's; card 7's refusal is the socket-path length.
I judged both against the shared fixture and gave the verdict on card 9, which owns the shell refusal and the fixture adjustments.
