```yaml
lens: way-forward
outcome: reported
```

Way-forward review: every refusal and recovery step the cards add has a next step, and each can succeed from the state it leaves. Nothing is BLOCKING; there are two LOW findings.

**Per refusal or recovery: the next step, and whether it progresses**
- **Card 1, a bad `run_watch`:** the existing `checkTaskSetting` message names `lym config beam --set run_watch=<value>` or the task's run_settings. Card 1 adds the key to the template, and run_settings writes go through `CheckLocalValues` (`internal/configreg/configreg.go:151`), so both routes take a valid value. Progresses.
- **Card 1, a negative `gate_hold_factor` or a `gate_hold_min` below 1:** the refusal names `lym config beam --set <key>=<value>`. The key is in the template, and an upper layer overrides a bad lower value. Progresses.
- **Card 3, a failed level read:** the last level is kept and the next tick reads again. Progresses once the read succeeds; see the first finding.
- **Card 3, no orch strand recorded:** the line waits without counting attempts and posts once a strand is recorded. The one-hour awake drop still bounds the wait. Progresses.
- **Card 3, a failing post:** it is retried after a probe interval and dropped after three failures, so the done-run clear it holds goes ahead. Progresses.
- **Card 3, an unreadable or unwritable state record:** one Warn, and the next tick tries again. Progresses.
- **Card 3, an entry whose batten status is gone:** owed lines still deliver, because delivery runs over every entry and not only the listed runs, and the entry is dropped once it owes nothing. Progresses.
- **Card 6, a failed config load, `Holders` read or `TaskWorktree`:** one Warn per episode, and the next probe tick checks again. Progresses.
- **Card 6, a failed post:** it is retried every probe tick while the hold stands, with no cap. A missing orch strand counts as a failure through `requirePosted` (card 7), and the retries end when the hold's key is dropped. Progresses.
- **Card 6, a failed `Tree` or `MarkedPIDs`:** the notice still posts and names the tree as unreadable. Progresses.

**Findings**
- **LOW, card 3 step 1 (D2), the failed level read:** the Warn is specified as "naming the full key `beam.run_watch`" but not as carrying the lookup error. That error holds the refusal's path, value and way forward. A task whose own layer holds a bad value never passes the dobby's own `LoadConfig`, so its watch stays silent indefinitely, and the Warn is the only signal. The Warn should carry the error, as `internal/dobbyengine/landnotice.go:84` passes `"error", err`.
- **LOW, card 6 "The step" (D6), the scheduling gate:** card 6 puts `d.gateHolds` inside the scheduling block (`internal/dobbyengine/loop.go:708-729`). An anchor move, an adopt hold or an owed catch-up then silences the gate-hold check for as long as it stands. A catch-up waiting on an unreachable origin is one case. D1 imposes that gate on the watch lines only; D6 says "On each probe tick the dobby reads the hub's slot holders" and names no such gate. The card either runs the step on every probe tick outside the scheduling block, or states the gate as a bound in its Intent.

Tool calls:
- Read `internal/boardengine/store.go:615-665` and grepped for callers of `CheckLocalValues`, to confirm that the run_settings route of the `run_watch` refusal's way forward accepts the new key. My context lacked the board's run_settings validation path.
