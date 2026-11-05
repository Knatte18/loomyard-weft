```yaml
lens: fix-way-forward
outcome: reported
```

Checked the ways forward the fix introduced in the decision record: the drop event and orch notice, the id-clash refusal, the final `deliver` before a return, and the `--reply` size refusal.

- **MEDIUM**: D3 step 6, "sends a `<shortname>:CREEL` notice of the drop to its hub's orch … naming the same fields and `lym creel log` for the drop".
  - **Summary:** the notice points the orch to `lym creel log` for the drop, but nothing makes a drop event show in `lym creel log`, and no flag finds it.
  - **Evidence:** `lym creel log` filters only by `--id` ("show only the events of the message with this id") and `--since` (`internal/creelcli/cli.go:762-765`). A drop event stores no message in the receiving hub, so it has no id for `--id`. D6 only says that log shows "the sending hub's path and id on the message's entry". No decision says that log renders the new drop event kind or how to select it, so the orch may not be able to reach the step.
- **MEDIUM**: D3 step 3, "an id clash, such as a hub re-created at the same path restarting its ids, and is answered `refused` with a reason naming the clash", and the matching D3 criterion.
  - **Summary:** an id clash returns the message to the sender, but no way forward is named, and the obvious one, re-posting, clashes again.
  - **Evidence:** D2 returns a refused message "undelivered at once" with a `CREEL` notice naming "the receiver, the reason and `lym creel log --id <id>`". Ids "restart from the largest id folded" from the journal (review F6), so a re-created hub hands out ids 1, 2, 3 and so on again. Its re-post gets the next low id, which also matches a held message from the old hub with a different tuple. Every new post is refused until the sending counter passes the receiver's held ids or they age out of the receiver's seven-day retention. Neither the refusal reason nor the notice says what to do: wait, or how to move the id range on.
- **No finding:** the drop notice's receiver `<shortname>:orch` can be reached. `checkReceiverLocked` accepts "the fixed name of an orch of this hub" even when no such strand is live (`internal/creelengine/pool.go:218-224`).
- **No finding:** the final `deliver` before a return (D2) ends in progress whatever it answers. `accepted` journals the message as delivered, and any other answer journals the return.
- **No finding:** the refusal of an over-cap reply at `--reply` (D4) leaves a clear next step, a shorter reply.

Calls made:
1. Grepped the `internal/creelcli` flags and the `internal/creelengine` receiver refusals. My context lacked the `lym creel log` flags and how creel treats a receiver that is not live.
2. Read `internal/creelengine/pool.go:215-240` and the notice functions. My context lacked whether `<shortname>:orch` is accepted as a receiver when no orch strand is live.
