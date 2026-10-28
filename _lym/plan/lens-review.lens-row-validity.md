```yaml
lens: row-validity
outcome: reported
```

Row-validity review of the 18-card creel-hook-post plan: whether each planned test row can reach its branch, would fail if the code were wrong, and is needed.

1. **MEDIUM — Card 10: the daemon rows cannot ring on the rig's strands.**
   - Card 10's new `TestDaemon_Protocol` row (a hook-wait gets rung by a post) and its `TestDaemon_Lifecycle` row (a hook-wait reconnects after a restart and is then rung) both need the receiver to carry `CreelHooks`.
   - The server rig gives every strand `creelfake.LiveStrand` (`internal/creelengine/server_test.go:72-75`).
   - After card 8, `LiveStrand` is `PaneMail: true` and `CreelHooks: false`, and card 10's `wakeHookStrand` runs only for a strand whose `CreelHooks` is set.
   - So neither row can reach `OutcomeRing` as written.
   - Fix: the card should say each row re-scripts alpha with `r.del.SetStrands(creelfake.HookStrand(alpha)…)` and the pane pid of the test process.
   - Card 13's CLI rows have the same gap: the hook-wait ring row and the Stop rows need a `HookStrand` receiver, and card 13's Uses does not list `plan:internal/testkit/creelfake#HookStrand`.

2. **MEDIUM — Card 11: the "ring answered within the window gets no bell" row cannot show the 30-second window.**
   - The bell's first branch fires when no waiter has been live for `bellGrace` (5 s) after the strand's last waiter ended.
   - A ring answers the waiter, and in production the waiter exits 2, so `Waiter.Cancel` records its end.
   - The bell therefore fires 5 s after any ring, before `ringAnswerWindow` (30 s) can matter.
   - The row passes only when the claim lands within 5 s; with a claim between 5 and 30 s it fails, or the row is vacuous.
   - The card never says whether a rung set is excluded from the grace branch, so the row's expected result is undefined.

3. **LOW — Cards 6 and 13: the Stop-count rows depend on message sizing the cards never state.**
   - Card 6 has "five more Stops ... each hand out one message, and the seventh claims nothing while mail still stands".
   - Card 13 has a "second Stop ... claims the rest".
   - One claim takes every unreserved message within the 8 KiB budget (`internal/creelengine/verbs.go:178-195`).
   - With small bodies, the first claim takes everything, so the later Stops claim nothing because no mail is left, not because of the cap.
   - Card 6's "while mail still stands" assertion then fails, or the cap branch goes untested.
   - Fix: state the bodies, for example near 5 KiB each so the budget admits one per claim, or one post per step.

4. **LOW — Card 8: the fake's stall branch has no stated `NoTurn`.**
   - `creelfake.Delivery.Idle` returns early on a scripted stall (`internal/testkit/creelfake/creelfake.go:204-206`).
   - Card 8 says only that `SetTurn` "makes Idle answer NoTurn: !running"; it does not say the stall branch carries it too.
   - Card 10's draft row needs exactly that: `SetTurn` running false plus a draft stall must ring.

5. **LOW — Card 7: the planned new row repeats an existing one.**
   - The row "held only for a draft reads `NoTurn` true with `Idle` false" duplicates "a draft in the box is held with its reason" (`internal/shuttleengine/session_test.go:279`), which the card already updates to want `NoTurn` true.
   - The card also leaves expectations unstated for two rows:
     - the operator-activity row (`session_test.go:291`): `SessionIdle` builds a fresh `IdleProbe{Reason: hold}` there, and the card does not say it copies `NoTurn` and `TurnEndedAt` onto it;
     - the unknown-reading rows (`session_test.go:277-283`): their expected `NoTurn` depends on whether `readinessEngine` implements `InterruptGuard`, which the card does not settle.

6. **LOW — Card 4: the new EXTERNAL-reply row repeats an existing row.**
   - The planned new row puts a 9 KiB reply to EXTERNAL through a waiting `PostWait`.
   - `Reply_ToExternalGoesToTheRegisteredWaiter` already covers that path (`internal/creelengine/verbs_test.go:291`).
   - The 9 KiB acceptance belongs as an assertion in that row; only the refusal of a strand's over-budget reply is new.

7. **LOW — Card 14: no tier for the new shell test.**
   - `TestCreelStopHook_RecordsATurnEndOnlyWithoutABlock` runs `sh`, as the tier1 tests in `hookcmd_integration_test.go` do.
   - The card names no tier or file for it; `PATTERN-test-speed` needs exactly one `tierN` tag.

8. **LOW — Card 2: wrong card reference.**
   - Card 2's Intent says "Claude's `Prepare` sets the field in card 13"; card 14 sets `Launch.CreelHooks`.

An aside for the guard-reach lens: during a woken turn, `Runner.SessionIdle` returns `Idle: true` when the hook reading is ready, even if the pane shows a turn running (`internal/shuttleengine/session.go:245-253`). Card 11's bell gates on `worker.idle`, so it may be typed into a woken turn.

Tool calls:
- `creelfake.go:139-152,305-335` and `grep ClaimAtBoundary`: the fake's interrupt locking and every test caller of the claim, which my context lacked.
- `session_test.go` test head: the location and fields of `TestRunner_SessionIdle_ReturnsScriptedClassification`, which I had not opened.
- `session_test.go` rows: the rows and engine that card 7 rewrites.
- `server_test.go` rig: whether a daemon can be restarted in a test, for card 10's reconnect row.
- `client.go:200-272` and `server_test.go:56-80`: how the client handles a pending answer, and the strands the rig scripts.
- `verbs_test.go` row names and the `shuttleengines` test list: whether the named rows and tests exist.
