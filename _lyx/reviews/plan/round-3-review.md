---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/00-overview.md:44
    summary: The D11 audit stops at the first call level and at the callback boundary, so per-tick Warns in reed's state load and in the shuttle readers that loop callbacks call have no D11 row and no card
    fix: >-
      Re-run the D11 sweep down the whole per-iteration call graph, including the readers a hook or seam callback calls, and add a row and a card change for each line it reaches.
      The sweep must at least add these rows.
      (1) `reedengine` `Engine.adoptPaneGenerationLocked` and `Engine.loadOrInitStateLocked`, reached on every watch tick through `reapplyLayout` and on every name-repair wake through `repairNames`, neither of which saves state: `could not read the live pane generation, leaving persisted pane bindings as they are` (guard), plus `persisted pane bindings were minted against a different tmux session incarnation, clearing them` and `cleared strand pane bindings that named a pane another owner already claims` (guard once per change of the recorded generation or cleared set, since the in-memory clear is reloaded from disk on every tick).
      Card 4 lists `internal/reedengine#Engine.adoptPaneGenerationLocked` and `internal/reedengine#Engine.loadOrInitStateLocked`, routes the read failure through `e.calleeFailures` on a `pane generation` key with a recovery line, records on `Engine` the last logged generation mismatch and cleared set so each logs at Warn once per change, and adds `TestCalleeFailuresLogOncePerEpisode` rows that fail and then heal the generation read.
      (2) `shuttleengine` `ReadAgentWaits`, `ReadAgentActivity` and `ReadSessionStates` (`skipping an unreadable run record`), `heldMarkerLive` and `ReadWaitMarkers` (`skipping an unreadable wait marker`) and `newestTurnEnd` (`could not parse a run's events for its activity`), reached on every wake of `shedverbs`' child watch through loomcli's `readAgentStates` (`Spec.Loop.OnWake` via `idleReleaseOn`), on every idle-watchdog wake through loomcli's `agentActivityFor`, and on every inner-run check through battencli's `readAgentActivity` and `readAgentStates` seams. These lines skip the record and return no error, so cards 9, 13 and 14's episodes never see them.
      Card 11 lists `internal/shuttleengine#ReadAgentWaits`, `internal/shuttleengine#heldMarkerLive`, `internal/shuttleengine#ReadAgentActivity`, `internal/shuttleengine#newestTurnEnd`, `internal/shuttleengine#ReadWaitMarkers` and `internal/shuttleengine#ReadSessionStates`, guards each line with a package-level `logger.EpisodeSet` keyed by the run directory or file path, which recovers on that path's next successful read, and extends an existing reader test with a row that fails one record's read and then heals it.
      (3) `battencli` `readSessionStatesByStrand` (`batten: session states unreadable; the notice logs carry none`), reached on every battenshed activity check through `readAgentActivity`: card 9 lists `internal/battencli#readSessionStatesByStrand` and guards it with an episode on a package-level value, with a test that fails and then heals.
      Also give `parentlock` `Store.withLock`'s `parentlock: release branch lock` Warn a verdict, since `Store.Acquire` reaches it on every poll.
      These three target additions together pass `lyx loom validate-plan --overlay` with no new blocking finding.
  - id: F2
    severity: BLOCKING
    class: scope
    location: _lyx/plan/00-overview.md:58
    summary: The keep verdict for repairSessionName's capture and type failures rests on a false premise, since both Warns repeat on every name-repair wake while the name still drifts
    fix: >-
      Change the row at `00-overview.md:58` so `could not capture pane for session name repair` and `could not type the session rename` read guard, card 3.
      In card 3, `repairSessionName` routes each through an episode on `Engine` (an `EpisodeSet` keyed by `capture:`+GUID and `rename:`+GUID, or two `Episode` fields), recovering on the next successful capture with `reed: session name repair captures the pane again` and on the next typed rename before the existing `repaired session name` Info.
      Extend `TestRepairNames_SessionName` with a row that fails `capture-pane` on two runs of the repair on one engine and then heals it, asserting one WARN, one DEBUG and one recovery INFO, and a row that does the same for the paste-submit failure.
      Keep `repaired session name` as a keep row.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/00-overview.md:53
    summary: The keep verdict for runResizePinsLocked holds only for a live box, while card 4 itself says a fallback box re-applies on every tick
    fix: >-
      Either change the `runResizePinsLocked` row to guard with a card 4 change (an `e.calleeFailures` key `pin:`+pane ID, recovering on that pin's next success, with a `TestCalleeFailuresLogOncePerEpisode` row that fails and then heals `resize-pane`), or restate its keep rationale so it covers the fallback-box case: with the live window size query failing, `applyLayoutLockedOpts` skips the box-equality guard, runs `select-layout` and the pins on every tick, and a failing pin then Warns on every tick.
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/plan/09-battenshed-read-guards.md:61
    summary: No named test reaches the recovery branch card 9 prescribes for reviewWaitNote, since the extended row fails ReviewWait on every check
    fix: >-
      In card 9, change the `TestInnerRun_RunningArmReviewWaitNote` extension so the failing row's `ReviewWait` fails on its first two checks and answers on the third (or add a fourth check), and assert one `level=WARN` `reviewer wait note unavailable` line, DEBUG lines after it, and one `level=INFO` `a repeated failure recovered` line naming it, the way the `TestInnerRun_StrandReadErrorSkipsTheReviveWithAWarn` extension already does.
---

### [BLOCKING] D11 audit misses per-tick Warns below the first callee and behind callbacks

**Location:** `_lyx/plan/00-overview.md:44` (the D11 table and its "Reaches" method)

**Issue:**
D11 requires every Info-or-above line in a loop or polled path to have a verdict, and the overview claims its sweep follows each loop's per-iteration call graph, hooks and callbacks included.
The sweep stops at the first callee level and at the edge of the callback's own package, so these sites have neither a row nor a card:

- `reapplyLayout` (`internal/reedengine/reapply.go:127`) calls `loadOrInitStateLocked` on every watch tick, and `repairNames` (`internal/reedengine/namerepair.go:110`) does the same on every name-repair wake.
  That calls `adoptPaneGenerationLocked`, whose `could not read the live pane generation` Warn (`generation.go:133`) fires on every tick while `display-message` fails.
  That is the same tmux verb whose failure card 4 guards in `liveBoxLocked`, so after card 4 the generation Warn is left as the unguarded flood.
  Neither path saves state (`reapplyLayout`'s doc comment says it never calls `SaveState`).
  So the in-memory clears behind `persisted pane bindings were minted against a different tmux session incarnation` (`generation.go:150`) and `cleared strand pane bindings that named a pane another owner already claims` (`spawn.go:363`) are reloaded from disk and logged again on every tick until some persisting op runs.
- loomcli's `readAgentStates` (`internal/loomcli/looplock.go:52`) is the agents func of `idleReleaseOn`, which is `shedverbs`' `Spec.Loop.OnWake`.
  It calls `shuttleengine.ReadAgentWaits`, which Warns `skipping an unreadable run record` (`agentwait.go:49`), and `heldMarkerLive`, which Warns `skipping an unreadable wait marker` (`agentwait.go:79`), per record on every wake.
  loomcli's `agentActivityFor` (`looparm.go:114`), the idle watchdog's agent func, calls `ReadWaitMarkers` (`waitmark.go:100`) and `ReadAgentActivity` (`activity.go:74`, and `newestTurnEnd`'s `could not parse a run's events` at `activity.go:168`) on every wake.
  battencli's `readAgentActivity` (`wire.go:554`) and `readAgentStates` (`parentlock.go:87`) are battenshed seams read on every check, and they reach the same readers plus `ReadSessionStates` (`sessionread.go:53`) and `readSessionStatesByStrand`'s `batten: session states unreadable` Warn (`wire.go:605`).
  Each of these readers logs and skips the record without returning an error.
  So cards 9, 13 and 14, which guard only the seam's returned error, leave them repeating with no state change.
- `parentlock` `Store.withLock`'s release Warn (`store.go:154`) runs on every poll of `Store.Acquire`'s wait and has no row.

This is one recurring gap in the method, filed once as `design`.
It is the third round in a row that finds it: rounds 1 and 2 each added the rows they named, but the sweep still stops one call level down and at the callback's package edge.
So the fix should come from a full re-sweep, not from these rows alone.
The focus directive's first bullet names this exact question.

### [BLOCKING] repairSessionName's capture and type failures repeat on every wake

**Location:** `_lyx/plan/00-overview.md:58`

**Issue:**
The row keeps `could not capture pane for session name repair` and `could not type the session rename`, on the ground that each is "retried only while the name still drifts and the pane is idle, and a repair ends the drift".
In `repairSessionName` (`internal/reedengine/namerepair.go:163`), the capture runs right after `SessionNameDrift` and before any idleness check.
So a failing capture returns on every wake for as long as the name drifts, and nothing changes the drift.
A failed paste-submit leaves the drift too, so the rename Warn also repeats on every wake while the pane reads idle.
Both are Warn lines that repeat without a state change, which D11's test forbids, and the keep rationale does not hold for either failure branch.

### [LOW] runResizePinsLocked's keep rationale ignores the fallback box

**Location:** `_lyx/plan/00-overview.md:53`

**Issue:**
The row keeps `failed to pin a pane height after the layout apply` because "a live box re-applies only when it changed".
`applyLayoutLockedOpts` (`internal/reedengine/apply.go:214`) takes the box-equality skip only when `boxIsLive`.
`handleWatchOutcome` leaves `lastApplied` untouched for a fallback box, and card 4's own Intent says the skip is never taken while the size query fails.
So during a live-box-query failure `select-layout` and `runResizePinsLocked` run on every tick, and a pin that also fails Warns on every tick.
The window is narrow, since two tmux calls must fail together, but the stated rationale is false for that case.

### [MEDIUM] reviewWaitNote's recovery branch is never reached by a test

**Location:** `_lyx/plan/09-battenshed-read-guards.md:61`

**Issue:**
Card 9 makes `reviewWaitNote` call `p.failures.Recover` on the seam's next success.
Its only test is the extended "failing ReviewWait falls back to plain reason" row of `TestInnerRun_RunningArmReviewWaitNote`, whose `ReviewWait` returns `errors.New("boom")` on every call (`internal/battenshed/innerrun_test.go:448`).
The card asserts only the one WARN and the later DEBUG lines there, and the new table's guard rows leave `reviewWaitNote` to that extension.
So no named test reaches the recovery line, unlike every other guarded site on the card.
The focus directive's third bullet asks for exactly this.

## Notes on the focus directives

- Reed per-tick Warns on the paths the second bullet names: layout reapply (card 3), live window size and malformed answer (card 4), border-title readback (card 4), zoom keeping (card 4), pane-title repair and the name-repair pass (card 4).
  Each has a guard verdict, and each card-4 row fails three calls and then heals, which reaches the WARN, DEBUG and recovery branches.
  The gaps on these paths are F1's state-load Warns and F3's pin rationale.
- Recovery conditions: card 11 states that a dismissal returning an error counts as a failure even when `sent` is above 0, and that only an error-free non-empty send recovers.
  Every other guard on cards 3, 4, 11, 12, 13, 14 and 15 recovers on a call that returns no error, and each names a test that fails and then succeeds, except F4's site.
- Every target fix proposed above (F1's additions to cards 4, 9 and 11) was overlaid together with `lyx loom validate-plan --overlay`, and the run reports no blocking finding beyond the `{}` baseline.
