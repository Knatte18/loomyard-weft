```yaml
lens: code-tests
outcome: reported
```

**Lens code-tests: per behaviour, the test that pins it, or none**

1. **Merged Stop command runs the turn hook before creel's delivery.**
   - Pinned by `TestBuildSettings_TurnHookCommand` (`settings_test.go`), which compares `wantMergedStop` exactly. Order, a single read of `in`, and the absence of a trailing hook step all fail it if broken.
   - Also pinned by `TestTurnHook_MergedCreelStopRunsItOnEveryTurnEnd`, which fails if the hook is skipped when mail is handed out.
   - Pinned by the row `TurnHookGetsTheInputWhenMailIsWritten` in `TestCreelStopHook_RecordsATurnEndOnlyWithoutABlock`, which checks the hook's stdin is the payload byte for byte.

2. **The turn hook's output never reaches the hook's stdout.** Pinned. Both stub verbs print `STUB_OUT`, yet the rows expect `mail+"\n"` once.

3. **The turn hook still runs when the turn end is recorded.** Pinned by the row `TurnHookGetsTheInputWhenTheTurnEnds` and by the second case of `TestTurnHook_MergedCreelStopRunsItOnEveryTurnEnd`.

4. **An empty `turnHookCommand` leaves the old behaviour.** Pinned. The rows without `turnHook` assert that no turn hook stdin file exists, and the output and recording assertions are unchanged.

5. **`creelOpenStopHookCommand` now opens with `in=$(cat); `.** Pinned by the three `Open*` rows. They run the stub and check the creel stdin.
   - `command_test.go:656` compares the function to itself, so it pins nothing.

6. **`afterEmptyPoll` returns 0 after a poll that lasted at least the floor.** Pinned by `EmptyPollThatWaitedOutTheFloorIsFollowedAtOnceByTheNext`, which advances the fake clock by `Ceiling` in `onReadWait` and expects 0 three times. A version that always called `afterIdle` would fail it.

7. **`afterEmptyPoll` backs off after a poll shorter than the floor.** Pinned by the existing test at `outbound_test.go:695-704`. The fake clock does not advance there, so elapsed is 0 and the waits double up to `Ceiling`.

8. **After a long poll the wait drops back to the floor (`w.current = floor`).** None. No test runs a long poll after backed-off waits and then a short one. It would fail if the line were removed.

9. **The `elapsed == floor` boundary, and a floor raised by a rate limit.** None. The test only uses elapsed `Ceiling` (60 s) against the 5 s floor. A change from `<` to `<=` or from `get()` to `waitFloor` would go unnoticed. NIT.

10. **The presence notice of slack skips the orch that was forwarded the operator's message.** Pinned by `OperatorsMessageThatTurnsThePresenceToSlackIsNoticedToEveryOrchButTheOneItWasForwardedTo`. Only `paymentsOrch` gets a notice, so the prime orch is skipped.

11. **A forward that fails, or a message that did not turn the presence, does not suppress the notice (`turned && err == nil`).** None. No row has the forward fail while the presence turns, or has a turn without an orch in the set.

12. **A notice of terminal ignores `awayFrom` (`presence != PresenceSlack → awayFrom = nil`).** None. The existing terminal-notice rows never populate `awayFrom` first. Removing that guard would still pass every test. A scenario where an operator's message turns the presence and the presence then goes back to terminal in the same or next wake is untested.

13. **`takeAwayFrom` clears the set even when the notice returns early (presence equals noticed, or nothing noticed yet).** None, so a stale entry could suppress a later notice without any test failing.

**Findings**

- **F-A, MEDIUM, design.** `internal/slackengine/relay.go:693` and `inbound.go:650`: no test pins the `presence != PresenceSlack → awayFrom = nil` guard, the `turned && err == nil` condition, or the clearing of the set by an early return.
  - Evidence: the only new relay row, `relay_test.go:991-1005`, covers the slack case with a successful forward. The terminal rows at `relay_test.go:944-960` never forward an operator message.
  - Add rows: (a) an operator message turns the presence to slack, then terminal; both orchs get the terminal notice. (b) A forward that fails (turned, err) still notices the orch. (c) Take-then-early-return does not carry an entry into a later notice.

- **F-B, LOW, design.** `backoff.go:74`: the drop of `current` to the floor after a long poll is not pinned. Add a short poll after backed-off waits and a long poll, and assert the next short poll waits `2*floor`, not a higher value.

- **F-C, NIT, consistency.** Test economy. The two new rows in `TestCreelStopHook_RecordsATurnEndOnlyWithoutABlock` (`TurnHookGetsTheInputWhenMailIsWritten` and `TurnHookGetsTheInputWhenTheTurnEnds`) overlap the behaviour `TestTurnHook_MergedCreelStopRunsItOnEveryTurnEnd` already covers (the turn hook runs whether or not mail is handed out). They add the payload-stdin and discarded-output assertions, so the overlap is partial. Fold the stdin check into the existing merged test, or keep the rows and say why.

**Tool calls**
1. `grep` in `outbound_test.go` and `relay_test.go` for notice, back-off and `ErrWaitTimeout` tests: the manifest did not hold the existing test names.
2. `grep` for wait assertions, plus `sed -n 940,975p` of `relay_test.go`: to confirm the doubling test at `outbound_test.go:695-704` and the terminal-notice rows. The first call's grep pattern missed these.
