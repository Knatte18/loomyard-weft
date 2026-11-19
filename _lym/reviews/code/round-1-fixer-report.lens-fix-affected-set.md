```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set searches behind F1, F3, F5 and F7 and check the fix covers every member.

**Findings**

1. **LOW**, F1 set (paths that put mail into the relay's mailbox), `internal/creelengine/verbs.go:415` and `Store.Forward`.
   - The review named `post` and `reply`, and the fix covers both. `Forward` also moves mail to an arbitrary receiver, and it carries no `slackRelayCheck`.
   - An orch whose button is off, or a non-orch strand, can run `lym creel forward <id> ly:SLACK` and get a plain ok. The relay then drops the mail, which is the original silent-success defect.
   - Evidence: the `Forward` body at `verbs.go` (the `checkReceiverLocked` then `deliverMailLocked` path) has no bridge call.
   - Every other `appendPostedLocked` site posts from a creel or program sender (`chase.go:212` and `:277`, `receive.go:390`, `outbound.go:182`). `slackRelayCheck` skips program senders, so none of them is affected.
   - This may be out of scope for the board entry. It is a member of the set, so the fixer should dispute it with a reason or defer it.

2. **LOW**, F3/F5 cross-reference, `internal/creelcli/cli.go:401` (the `post` help text).
   - It says "answers receiver_live, and one not live gets a warning". It does not mention `delivered` or `post --reply` to relay mail.
   - The help is size-capped (commit `fe73fb3e`), and `creelcli/doc.go` was updated, so this is a stale-sibling note and not a defect.
   - A cut elsewhere may be needed to add a few words.

**Searches that found no gap**
- F3: every `Response` build that carries `ReceiverLive` also carries `Delivered`. That covers the plain post, the reply, the pending response and the final waiting response (`server.go:316`, `:320`, `:332`, `:472`, `:508`). The CLI reads both in `addReceiverState`. `server.go:432` `req.Delivered` is an unrelated request field.
- F5: no other source or doc contains "until the relay starts".
- F7: no other contract or doc says relay mail is "dropped while the button is off" in the sense the role file did. `slackengine` docs describe the relay's own behaviour, which is unchanged.
- F4/F6: the only `RelayPostReady` implementations are `slackBridge` and the `creelfake.Bridge`. Both are covered.

**Calls**
1. A grep for `ReceiverLive`, `Delivered` sites, stale relay text and `appendPostedLocked`/`slackRelayCheck` sites across `internal`, `contracts` and `docs`. My context lacked sites outside the files the fixer edited.
2. A read of `cli.go` lines 396-408 and a grep for other consumers of `receiver_live`, plus `crossmachine.go` relay mentions. My context did not show the help text after the edit or any consumers outside the engine and CLI.
