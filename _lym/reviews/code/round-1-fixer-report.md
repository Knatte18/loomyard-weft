---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed-differently
    reason: "Did not add a per-cgroup named count beside the kill count. The told count already is the journal offset, since the journal lists a cgroup's kills in the order the counter counts them, so the notice names killed[told:told+newKills] and falls back to the count-only line when the journal holds fewer. A separate named count would have to advance by newKills on a count-only notice anyway, so it equals told."
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed-differently
    reason: "Did not return at the first failed or inactive read, and did not read the unit's Result. A scope whose start job is still queued reads inactive like a dropped one (busctl get-property on an absent unit answers inactive), so the wait gives up only after two ended reads in a row, the reads injected as a function so a test drives them. The Result is not read because the scope's collect mode drops a failed unit at once and its Result is then gone; the error names the state."
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
---

# Round 1 fixer report

## Changes

Commits, one per group of findings that touch a common file:

- `reed: fix F1, F2, F3, F4, F5, F7, F8, F9` (`041679ab8`), with a self-check follow-up `fda7206a6`:
  - F1: each entered session starts its own `watchPaneOOMKills` goroutine in `enterSession`, `watchdog: off` sessions included, with its own backed-off cadence from `watchdogTiming`.
    The discovery loop no longer reads the strands, the journal or posts.
    `watchedSession` drops `location` and `oomKills` and gains `repairsNames`; its `cancel` is always set, so the loop's nil checks are gone.
    The `runWatchdogLoop` doc, the `enterSession` doc and the package doc describe the watch.
    The two `enterSession` integration tests pass a timing and assert `repairsNames` instead of a nil `cancel`.
  - F2: `reedengine.PaneScopeUnitPrefix` is exported, and `planPaneOOMNotices` skips a pane whose cgroup base name does not start with it.
    `TestPlanPaneOOMNotices` has a pane in a `tmux-spawn-` scope holding kills and asserts no notice.
  - F3: a notice carries `told`, and `paneOOMNoticeLine` names the entries after the kills already told.
    A new `TestPaneOOMNoticeLine` row holds a journal with only the told kill and asserts the count-only line.
  - F4: `paneOOMBoundaries` holds the strand reader, the kill reader, the journal reader and the poster; `engineOOMBoundaries` wires production.
    `TestReportPaneOOMKills` drives a failed post, its retry, a repeat read and a lagging journal through fakes.
    `TestWatchPaneOOMKills_PostsANewKillOnceAndStopsWithItsContext` drives the watch loop.
  - F5, F9: `panememory_test.go` holds `TestLaunchStrandLocked_APaneThatCannotBeCappedStillLaunchesWithAWarning`, with rows for an unreadable, a non-numeric and a zero shell pid, and `TestPaneMemoryMax`.
  - F7: `planPaneOOMNotices` leaves `seen` as it was, and `reportPaneOOMKills` advances it after `post` succeeds.
  - F8: the busctl spawn logs at Info.
- `proc: fix F6` (`2dc85b9d5`), with a self-check follow-up `d122168e0`:
  `waitForScope` reads the scope's `ActiveState` through busctl every tenth poll and returns an `ErrMemoryCap` error naming the state after two ended reads in a row.
  `TestWaitForScope_GivesUpOnAScopeReadEndedTwiceInARow` drives it with a fake state reader; `TestUnitObjectPathLabel` and `TestScopeStateEnded` pin the escape and the parse.
- `proc: fix F10, F12` (`d783f61e7`), with a self-check follow-up `fcc1f21b4`:
  the capped-process test reads `memory.swap.max` and asserts 0, and skips that read on a host with no such file.
  The ended-process test asserts the `refused the scope` text.
- `reedengine: fix F11` (`b553cec62`): `pane_memory_max_mib 8192` is in the keep reason.

A gate pass then broke the wrapped and over-long comment lines the round added in `oomnotice.go`, `oomnotice_test.go`, `watchdog.go` and `export_linux_test.go` into one sentence per line, with no change to code.

A second gate pass found internal/proc code no tier0 test reached.
It added tier0 tests for `scopeEnded` with no busctl, `ProcessOOMKills` of the test process, and `CapProcessMemory` and `OOMKillsInCgroup` with an empty PATH.

The tier0 run of `internal/reedcli`, `internal/reedengine` and `internal/proc`, and the tier1 run of `internal/proc` and `internal/reedcli`, pass.

## Self-check

- fix-consistency, LOW, `internal/reedcli/doc.go:7` still says each cycle reads the kill counts: fixed.
- fix-consistency, LOW, `entersession_integration_test.go` comment says no watcher: fixed.
- fix-consistency, LOW, `watchdog_integration_test.go` comment says no watcher: fixed.
- fix-affected-set, LOW, the same three stale texts: merged with the fix-consistency findings, fixed.
- fix-code-correctness, MEDIUM, a queued scope reads inactive and fails at 200 ms: fixed differently, as the F6 entry says; the fix-proposal lens raised the same case.
- fix-code-correctness, LOW, no test of the early return in `waitForScope`: fixed with `TestWaitForScope_GivesUpOnAScopeReadEndedTwiceInARow`.
- fix-code-correctness, LOW, the watch test's duplicate check can pass with the defect in place: fixed; the test now waits for further reads after the post before it cancels.
- fix-code-correctness, LOW, compressed timing raises op-lock contention in integration runs: disputed.
- fix-proposal, MEDIUM, the F6 queued-scope case: merged with the fix-code-correctness finding.
- fix-proposal, LOW, F7 retries a permanently refused post: disputed.
- fix-proposal, LOW, F3's told offset drifts when a pane makes a child cgroup: disputed.
- fix-proposal, LOW, F10's swap read fails on a host without swap accounting: fixed.
- fix-proposal, LOW, the watch goroutine is not in the daemon's wait group: disputed.

## Disputed

- fix-code-correctness LOW, compressed timing and op-lock contention.
  Ground: premise false.
  The lens names no failure, and production passes the fixed constants through `watchdogDefaultTiming`.
  The tier1 run of the integration tests with the compressed timings passes (`internal/reedcli/watchdog_integration_test.go`).
- fix-proposal LOW, F7 retrying a refused post.
  Ground: every fix worse.
  The alternative, advancing `seen` before the post, is the review's F7 premise: it loses the kill for good (`internal/reedcli/oomnotice.go:126`).
  The retry is bounded to one attempt per `watchdogHubDiscoveryCeiling` once posts fail, with one Warn each (`internal/reedcli/watchdog.go:40`).
- fix-proposal LOW, F3's offset when a pane makes a child cgroup.
  Ground: every fix worse.
  The review's own proposal and the code before the fix share the flaw, and no sound alignment of the `memory.events` counter with the journal entries of one cgroup exists without reading descendants' journal entries, which `parseOOMKills` deliberately leaves out (`internal/proc/processscope_linux.go:166`).
  The effect is a count-only line or one wrong name, never a lost notice.
- fix-proposal LOW, the watch goroutine is not waited for at daemon exit.
  Ground: premise false.
  The daemon never waited for the name-repair goroutine either, and nothing relies on either ending before the process does (`internal/reedcli/watchdog.go:362`).

## Deferred

Nothing deferred.

## Rulings

No ruling reached this round.
