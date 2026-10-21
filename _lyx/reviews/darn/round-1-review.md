---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/reedcli/watchdog.go:516-523
    summary: the unwatched timer arm Warns on every wake while consumeDiscoverSignal keeps failing, against PATTERN-log-levels
    fix: |-
      In runWatchdogLoop declare `var consumeFailure logger.Episode` beside `unwatched`. In the timer arm replace the `logger.Warn("reed: watchdog could not consume the discover signal", "hub", hub, "err", err)` with `consumeFailure.Fail("reed: watchdog could not consume the discover signal", "hub", hub, "err", err)`. In the same arm's non-error path call `consumeFailure.Recover("reed: watchdog consumes the discover signal again", "hub", hub)`, which does nothing outside an episode. Leave the events-branch Warn as it is.
    origin: lens:code-correctness
  - id: F2
    severity: LOW
    class: consistency
    location: internal/reedcli/watchdog.go:524-530
    summary: the closed-event-channel path that sets unwatched and Warns has no test
    fix: |-
      Add a row to TestWatchdogCmd_ExitsZeroBeforeOrInsteadOfLooping in internal/reedcli/watchdog_test.go. Its opener returns an already-closed channel instead of an error, via a new `closedWatch` field next to `failWatch` that sets `c.watchdogOpenWatch`, and it uses the same lister and stop write as "stop with an idle listing and no watch". Assert the trace lines "watch closed, reading the file on its backed-off cadence instead" and "exiting on a stop request".
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: consistency
    location: internal/reedengine/naming_integration_test.go:69
    summary: a comment still says spawn warns on tmux 3.4 for allow-set-title, but spawn now skips the option silently on a parsed version below 3.5
    fix: |-
      Reword the comment on line 69 to say spawn skips the allow-set-title option on a tmux older than 3.5, so the check applies only where the option exists. Leave the code of the switch below it unchanged.
    origin: lens:code-affected-set
---

### [BLOCKING] Unwatched timer arm Warns on every wake while the consume keeps failing

**Location:** internal/reedcli/watchdog.go:516-523
**Issue:** The new timer arm calls `consumeDiscoverSignal` on every wake while `unwatched` is set, and logs `logger.Warn` for each error.
A persistent fault, such as a signal file that cannot be read or removed, repeats on every tick, so the daemon logs one Warn per cycle for as long as the fault lasts.
`PATTERN-log-levels` points at `internal/logger`'s Level policy: a failure that repeats each tick logs Warn once per episode, Debug while it persists and Info at recovery, and `logger.Episode` implements that.
The events-branch Warn the arm copies fires once per event, so it is not the same flood.

### [LOW] Closed-event-channel fallback has no test

**Location:** internal/reedcli/watchdog.go:524-530
**Issue:** The diff adds a behavior: when the event channel closes, the loop sets `unwatched` and Warns, so later timer wakes read the signal file.
The new row drives only a failing open, and the other rows and `TestWatchdogDaemon` use a working or never-delivering watch, so no test closes a channel and checks that a stop is still read.
The decision record's scope names the closed channel as one of the two triggers.
This is the rubric's "a behavior the diff adds with no coverage".

### [LOW] Stale comment says spawn warns on tmux 3.4

**Location:** internal/reedengine/naming_integration_test.go:69
**Issue:** The comment reads "Older tmux (3.4 included) has no allow-set-title option; spawn only warns then".
After this diff, a launch on a tmux whose version line parses below 3.5 skips the option and notes it once at Info, and Warns only on an unparsed line.
The code below the comment stays correct.

## Rejected

- `tmuxkit.ServerEnviron` keeps its `"LYX_TRACE_ID"` literal (code-pattern-fit, informational): the board entry names only the three test files, the summary records the departure from the decision record, `go list -deps` confirms the cycle (`logger` reaches `dotgit`, whose in-package `TestMain` imports `tmuxkit`), and `TestEnvScrub_MatchesTmuxkitServerEnviron` now fails on a rename that leaves the literal behind.
- A `floorLine` row of `tmux next-3.6` for the hook branch of `lacksAllowSetTitle` (code-tests, NIT): the unparsed case is already pinned in `TestTmuxHasAllowSetTitle` and the empty-line row of `TestLaunchStrandLocked_SplitWindowAndPaneTitleCalls`, so the row adds no behavior.
- `TmuxWithAllowSetTitleSetsIt` overlapping the empty-`floorLine` rows (code-tests note): it adds the parsed at-or-above input, so it is not a redundant test.
- The Info note logged for a parsed version below the 3.4 floor (code-correctness note): the floor refusal comes first, so the launch never runs.
- Focus questions settled with no defect: a stop marker is read once and cannot be lost between the timer read and `onSignal`, since both run in one select arm; the arm adds no timer reset, so the backoff and the discovery ceiling are unchanged; `tmux 3.5a`, `tmux 3.10` and `tmux next-3.6` parse as at-or-above, at-or-above and unparsed; the floor cache mutex spans the probe, so the Info note is logged once per path with no race; the summary's claims match the diff, and the `doc.go` updates for `toolfloor`, `reedengine` and `reedcli` land in it.
