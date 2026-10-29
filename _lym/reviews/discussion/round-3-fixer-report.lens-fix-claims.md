```yaml
lens: fix-claims
outcome: reported
```

fix-claims: I checked the claims the fix added to D5, D6 and the D6 acceptance criterion against the code at HEAD, and they hold.

no findings

- **The handed-out echo is printed before the acknowledgement.** The new D6 sentence says `hook-read` "printed it before the client acknowledged", and the code agrees. In `internal/creelengine/client.go:253-264`, on `AckRequired` the client calls `handle(resp)` first, then writes the acknowledgement line, then calls `awaitAckConfirmation`. `hook-read` passes `writeBlock` as `handle`, and `writeBlock` writes the output to stdout (`internal/creelcli/hookread.go:110-126`).
- **A failed acknowledgement leaves a journaled prefix.** The new D6 ring sentence speaks of "the ids the daemon journaled read before the failure", and `Reading.Ack` (`internal/creelengine/verbs.go`) produces exactly that. It journals the ids in order and returns at the first failing `journalReadLocked`, so the ids before the failure are journaled read and the ones after are not. Releasing those later ids is D3's design change, which the sentence cites rather than claims as current behaviour.
- **Mail the daemon did not journal reaches the operator at a later claim.** This follows from D1 and D2: a later tool-boundary claim puts the lines in `systemMessage`, and a later Stop claim shows them in the `Stop hook feedback:` summary row. D1 already states the 5-line limit on that row, so the new sentence adds no bound of its own.
- **The D5 and acceptance wording adds no new fact.** The rewrite of `ack-failed` and the new criterion only restate the D6 rule above.

Tool calls:
- `grep` of `internal/creelengine/client.go` for the handler and acknowledgement sites: my context did not have the client's handler-then-acknowledge order.
- `sed -n 248,270p internal/creelengine/client.go`: to read that branch in full and confirm the order.
