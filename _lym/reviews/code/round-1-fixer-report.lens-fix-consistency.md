```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix change set for consistency with its surroundings.

**Findings**

1. **MEDIUM** — `internal/creelengine/doc.go:105-110`, the package doc paragraph on the relay exception.
   - It says the exception is for "a post from a strand or EXTERNAL" and that "the post answers whether the relay runs".
   - It never mentions `delivered`, the new `post --reply` check, or the one-day limit on a waiting post.
   - Evidence: grep shows this paragraph untouched by the change set. `ReplyWith` now runs `slackRelayCheck` and carries `Delivered`.
   - A reader of the package doc still learns that a reply to relay mail is unchecked.

2. **LOW** — `internal/creelengine/crossmachine.go:19`, the `MachineBridge` doc.
   - It says "a daemon [without the bridge] stores a post to this hub's Slack relay program without asking anything."
   - `RelayPostReady` (line 27) is now also called from the reply path.
   - Its doc comment says "for a post from sender to the relay's program" and does not cover a reply.
   - The doc is not wrong, but it is stale next to the new caller.

3. **LOW** — `internal/creelcli/cli.go:401-403`, the `post` help text.
   - It says a post to the relay "answers receiver_live".
   - It omits `delivered` and the reply behaviour, which `creelcli/doc.go:47-48` now documents.
   - The 2000-byte help cap may block the addition. Even so, the two documents now disagree on the fields.

4. **LOW** — `internal/creelengine/protocol.go`, the `Response.Warning` and `Warning` comments.
   - "Warning names a post's receiver that is not live or running" still covers only a post.
   - A `VerbReply` response now also carries `Warning`, `ReceiverLive` and `Delivered`.
   - The `ReceiverLive` comment likewise says it answers "a post".
   - The `Delivered` comment was updated ("post or reply"), so the sibling comments are inconsistent.

5. **LOW** — `internal/creelengine/verbs.go`, the `PostResult.ReceiverLive` and `PostResult.Warning` comments.
   - `PostResult` is now also the result of `ReplyWith`.
   - The comments still describe only a post to a receiver.
   - `Delivered` was updated; the others were not.

**Checked and consistent:** the `relayDown` constant in the test matches `relayDownWarning`. The `role-slack.md` wording matches the new refusal and warning. `summary.md` matches the code. The `Reply` wrapper's callers are unaffected.

**Tool calls (2):**
- `grep` of `SLACK|relay` in `creelengine/doc.go` and `crossmachine.go`, plus `sed` of `cli.go:396-412`, plus `grep -l receiver_live`: my context lacked these sibling docs and the help text.
