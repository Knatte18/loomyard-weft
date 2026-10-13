# Fixer report — plan, round 3

## Changes

- **F1 (fixed, partly differently).**
  The D11 method paragraph in `00-overview.md` now follows the call graph to its leaves, into every package a hook, callback or seam calls.
  It also states that a line which logs and returns no error gets its own row.
  The re-sweep followed the reed watch tick and name-repair wake, the loomcli wake and idle-watchdog funcs, battenshed's `InnerRunDeps` seams as battencli implements them, and the orch watcher's shuttle seams.
  It added these rows:
  - (1) Reed: rows for `adoptPaneGenerationLocked`'s generation-read Warn (guard), its stale-stamp clear and `loadOrInitStateLocked`'s conflicting-binding clear (Warn once per change, Debug while it repeats), all on card 4.
    Card 4 lists both methods and `TestLoadOrInitStateLocked`.
    It guards the read on `calleeFailures` key `pane generation` with a recovery line and adds a `TestCalleeFailuresLogOncePerEpisode` row that fails and then heals the probe.
    It records the last logged generation pair and cleared set in a mutex-guarded `stateLoadNotices` value on `Engine`, and extends `TestLoadOrInitStateLocked` with two rows that load three times unsaved and assert one WARN and two DEBUG lines.
  - (2) Shuttle: rows for the run-record, wait-marker and events-parse skip lines, all on card 11.
    Card 11 lists the six readers and guards them with a package-level `unreadableFiles logger.EpisodeSet` keyed by path, with recovery lines on the next successful read.
    It extends `TestReadAgentWaits`, `TestReadAgentActivity`, `TestReadWaitMarker` and `TestReadSessionStates` with fail-three-then-heal subtests.
    `TestReadSessionStates` drops its `t.Parallel` so it can capture the logger.
    Each reader gets its own subtest because each reader is its own call site.
  - (3) Fixed differently: `readSessionStatesByStrand` gets a keep row instead of a card 9 guard.
    The Warn cannot repeat on its own: `ReadSessionStates` errors only on the run root's `ReadDir`.
    `readAgentActivity` (`internal/battencli/wire.go:560-565`) has just read that same root without an error through `ReadAgentActivity`.
    A root that stays unreadable therefore fails that earlier call, whose returned error card 9 already guards at `agentActivity`.
    A guard and a test here would cover a line that only a one-tick race reaches.
  - `parentlock` `Store.withLock`'s release Warn gets a keep row.
    gofrs/flock v0.8.1's `Unlock` returns a failed `LOCK_UN` before it closes the descriptor.
    The flock therefore stays held, and the next poll's blocking `AcquireWriteLock` waits on it instead of reaching the line again.
  - The sweep found one more site the review did not name.
    battencli's `childGoto` and the `Spawn` closure in `battenCLI.wire` each log two Info lines on every call, and battenshed calls them on every Publish-retry and resume retry.
    New overview rows demote them to Debug on card 9, which now lists `internal/battencli#childGoto` and `internal/battencli#battenCLI.wire`.
    battenshed's own Info-on-first-attempt lines around each call keep the attempt record.
    No test asserts these lines, and both seams exec `lyx`, so the card adds no test.
  - `battencli` `lockIdleBound`'s Warn gets a keep row, since it runs once per `wire`.
- **F2 (fixed as proposed).**
  The `repairSessionName` row is split: capture and type failures are guard, per strand, on card 3, and `repaired session name` stays a keep row.
  Card 3 adds `Engine.nameRepairFailures logger.EpisodeSet` keyed `capture:`+GUID and `rename:`+GUID, with recovery lines.
  It adds two `TestRepairNames_SessionName` rows that fail `capture-pane` or `load-buffer` three times and then heal, asserting one WARN, two DEBUG and one INFO.
- **F3 (fixed with the first option).**
  `runResizePinsLocked` is now guard, per pane, on card 4: `calleeFailures` key `pin:`+pane ID with a recovery line.
  The new `TestCalleeFailuresLogOncePerEpisode` row "pane height pin" fails `resize-pane` and then heals it.
  Card 4's Intent states the fallback-box reason.
- **F4 (fixed as proposed).**
  The extended row in `TestInnerRun_RunningArmReviewWaitNote` now fails `ReviewWait` on its first two checks and answers an empty note on the third, captures at Debug, and asserts one WARN, one DEBUG and one `a repeated failure recovered` INFO line.

`lyx loom validate-plan` reports `ok: true` with informational findings only.

## Disputed

None.

## Deferred

None.
