```yaml
lens: fix-way-forward
outcome: reported
```

Lens fix-way-forward: I checked the refusals, next steps, bounds and acceptance rows that this fix added to `decision-record.md`.

Ways forward the fix introduced:
- D10: a reply to a chase from anyone other than the holder is refused, with the way forward "post to the original sender by name".
- D10 Bound: a waiting `--wait-reply` sender is "bounded only by its own call timeout".
- D3: the hold notice says to answer with `lym creel post --reply <id>` when the marked question is in the orch's mail, and otherwise to post to the strand.
- D5: the orch answers a hold notice with `--reply` when a marked question is pending.
- D3: the burler, loom and webster wirings move to batten's run-notice rule.
- D10: a holder found gone is settled `holder-gone` through the widened sweep.
- New acceptance rows: D3, D5 and the expanded D10 row.

**MEDIUM — D10, the refusal of a non-holder's reply to a chase.**
The way forward "post to the original sender by name" can't be carried out when the original sender is EXTERNAL. When the sender is a program, the post leads nowhere.
- `internal/creelengine/pool.go:200`: `checkReceiverLocked` refuses any EXTERNAL name as a receiver ("is never a receiver: it has no address").
- The fix's own new sentence makes the chase name an EXTERNAL sender "by the bare EXTERNAL name, without its envelope", so the forwardee has no address at all.
- A program sender, such as `LOOM`, passes `checkReceiverLocked` through `s.state.programs[name]`. The post is then stored in a mailbox nobody reads, which is the same dead end that F5's premise names for `CREEL`.
- The refusal needs a way forward per sender kind, for example asking the holder of the chased message or, failing that, the forwardee's parent with `--parent`. The alternative is to state that chases can't be forwarded.

**LOW — D10 Bound, "A waiting `--wait-reply` sender ... is bounded only by its own call timeout".**
That reads as if a bound exists, but the default call waits forever.
- `internal/creelcli/cli.go:386`: `--timeout` defaults to `0`, "0 for never".
- The sentence should say the wait is unbounded unless the sender passes `--timeout`.

Nothing else I checked is a finding:
- The D3 and D5 `--reply` route can be carried out: the orch holds the question once D1 and D3 resolve to the same orch, and `Store.Reply` accepts a reply from the holder of unread mail (`verbs.go:194`).
- The D3 wiring change, the holder-gone settlement and the new acceptance rows can all be reached.

Tool calls beyond my context:
1. Grepped `verbs.go` and listed `creelcli`, to find the post receiver checks and the wait timeout, which my context lacked.
2. Grepped `storePost` and `cli.go`, to find the receiver check site and the `--timeout` handling.
3. Read `checkReceiverLocked` in `pool.go` and the `--timeout` flag default in `cli.go`, to confirm whether EXTERNAL and program receivers are reachable and whether the wait is bounded by default.
