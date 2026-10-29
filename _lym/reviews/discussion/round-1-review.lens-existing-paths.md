```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths: what D1–D5 meet in the code that already runs.

**Paths traced, per decision:**
- D1: merged Stop command `… hook-read … && exit 0; fi; record` (`settings.go:146-150`). Survives. Exit 0 after output still skips the turn-end record, and exit 1 on nothing handed out still records it.
- D1: `ClaimAtBoundary` Stop count (`verbs.go:325-353`) on a continuation whose next Stop carries `stop_hook_active` true. Survives. The count goes up, is not reset, and the cap of 6 holds.
- D1: `wakeHookStrand` idle and `TurnEndedAt` reads (`deliver.go:257-290`). Survives. A continued turn records no turn end, the same as a block does today.
- D2: tool-boundary claim resets `stopBlocks` and marks the ring answered on hand-out (`verbs.go:343-352`). Survives. Only the output shape changes.
- D3: `ringHook` claimable-set check and `wakeMarks` record (`verbs.go:435-454`). Survives, provided the ring's claim skips the hand-out `answered` marking at `verbs.go:344-347`, which D3 states.
- D3: ring then `resetStopBlocks`, with the woken turn's first Stop carrying `stop_hook_active` true. Survives. The asyncRewake notification is enqueued with `stopHookActive:!0` (binary, `_tt`), and the ring's reset leaves the count known at 0.
- D3: mail flag sync (`flags.go:17-25`, `pool.go:156-163`). Survives. The flag follows unread status, not reservation, so mail left past the budget keeps its flag.
- D3: release after a failed ack, then the bell (`deliver.go:259-290`, `Reading.Release` `verbs.go:489`). Survives. The claimable set equals `mark.ids`, so there is no re-ring, and the bell fires after `ringAnswerWindow`.
- D3: leftovers past the budget during the woken turn. Survives. `typeBell` needs an idle strand, and the turn's tool or Stop claim takes them.
- D3: `HookWait` supersede (`verbs.go:409-411`, `answerLocked` `pool.go:377-384`). Survives. An answered waiter is already off the key, so a later waiter cannot supersede a ring awaiting its ack.
- D3: daemon `awaitOutcome` exit paths (`server.go:388-422`). **Breaks**, see finding 1.
- D4: `hookEntry` (`settings.go:69`). Survives. There is no readback or strict decode of the rendered settings in claudeengine.
- D5: release logging on the ring path. Depends on finding 1. There is no existing release site on the ring path to log.

**Finding 1**
- Severity: MEDIUM. Class: `design`.
- Location: decision-record D3, the reservation held "until the client acknowledges them on the same connection", and its acceptance bullet "a connection dropped before the acknowledgement leaves the mail unread".
- Summary: the record is silent about the existing hook-wait answer path, which can drop a ring outcome without ever reading it. Under D3 that outcome holds a reservation that nothing releases, so the mail is stuck until the daemon restarts.
- How it breaks:
  - `answerLocked` puts the outcome into the waiter's 1-buffered channel and deletes the waiter (`internal/creelengine/pool.go:350`, `pool.go:377-384`).
  - `awaitOutcome`'s `select` can take `watcher.closed` or `d.done` instead of `waiter.C` and return with no further handling (`internal/creelengine/server.go:401-420`). This happens when the waiter client leaves, for example when Claude Code ends the hook at session end or timeout, as the ring is answered.
  - Today the outcome carries nothing, so that is harmless. Under D3 the reservation lives in the in-memory `reserved` map: `ClaimAtBoundary` sets it (`verbs.go:342`), and only `Reading.Ack` and `Reading.Release` clear it (`verbs.go:471-500`).
  - The stuck mail is excluded from claims and from `hookWakeState`, so no tool claim, Stop claim, ring or bell ever reaches it, while its flag keeps standing.
  - D5's `released` outcome also has no site there.
- Fix: D3 states that every exit of the ring's answer path without a confirmed ack releases the claim: connection closed, daemon shutdown, write failure, and an outcome left in the channel after the waiter was cancelled. D5 logs that release.

Tool calls:
1. `settings.go` `creelStopHookCommand` and `creelWaitHookCommand`: the merged Stop command's exit-code rule, which D1 relies on, was not in context.
2. Grep for flag writers in creelengine: the flag that D3's leftovers stand on had no definition in context.
3. `flags.go` header, `pool.go` `do`, `Reading.Ack` and `Reading.Release`: how the flag and the reservation are kept, which the manifest lacked.
4. Settings readers, and the `Waiter.Cancel`, `addWaiterLocked` and `answerLocked` bodies: whether a D4 readback exists, and how a ring outcome reaches `awaitOutcome`.
