```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each changed signature, output, message and behaviour I checked what reads or describes it outside the lines the diff touches.

**Per change, the set found and whether it is covered:**

- **`MachineBridge.RelayPostReady`, a new interface method.**
  - Implementers: `creelcli.slackBridge` (`slackbridge.go:96`) and `creelfake.Bridge` (`bridge.go:53`). Both are updated, and the repository has no other implementer.
  - Production wiring: `server.go:97` passes `opts.Bridge`, which comes from `newSlackBridge`. Covered.
  - `cli_test.go:295` `readySlackBridge` builds a `*slackBridge`, which gains the method automatically. Covered.
- **`creelfake.Bridge.Refuse`, now keyed for both `SendReady` and `RelayPostReady`.**
  - Existing callers `verbs_test.go:440,448` and `machinemail_test.go:417` refuse an orch for machine-address posts and replies, never for a post to `ly:SLACK`.
  - The shared map has no cross-effect and those tests keep their meaning. Covered.
- **Post to `<shortname>:SLACK` now refused for non-orch senders, orchs without a button and orchs with the button off; stored with `receiver_live` and a warning otherwise.**
  - `creelengine/doc.go:104-110`, `creelcli/doc.go:47,146-150`, `protocol.go` `ReceiverLive`/`Warning` and `verbs.go` `PostResult` are all updated. Covered.
  - `slackengine/doc.go:82` ("drops the mail of an orch whose button is off") stays true on the relay side, because a button can turn off between the post and the relay's read. Covered.
  - `slackcli/hook.go:176-195`, the turn hook, posts to SLACK as the orch strand only after it checks `ForwardingOn`. It passes the new check, and a down relay gives a warning, not an error. The `hook_test.go:95-110` fakes the client. Covered.
  - `PATTERN-agent-addressing` ("an agent reaches Slack only by creel mail to that program") still holds. Covered.
  - **`contracts/orchrole/role-slack.md:7` is not covered.** It still tells the orch that "mail you post to `<shortname>:SLACK` is dropped" while its button is off. Such a post is now refused with a way forward and nothing is stored. Line 16 sends the orch to post to SLACK and does not prepare it for a refusal or a `creel: warning:` about a down relay.
- **`postCmd` Long text** (the lines it removed and the SLACK lines it added).
  - Idle wake: still described in `creelcli/doc.go:90`.
  - Expects-reply mark beside the reply command: `creelengine/doc.go:202` and `creelcli/doc.go:22`.
  - `--parent` `receiver` field: `creelcli/doc.go:72`.
  - The package docs still carry each removed topic, so no doc is left wrong by the removal; whether the help text should drop them is the handler's or scope's question.
  - `cmd/lym/clitree_test.go:169` caps Long at 2000 bytes; commit `fe73fb3e` trimmed to fit it. Covered.
- **Rule documents and commits.**
  - The range has no `Pattern-Departure:` or `Loosened-Check:` trailer, and none is needed: no listed PATTERN entry forbids this change, and no test check is loosened.
  - No PATTERN changes are declared, so there are none to land.

**Findings:**

1. **MEDIUM**, consistency, `contracts/orchrole/role-slack.md:7`. The orch role's Slack file still says mail posted to `<shortname>:SLACK` while the button is off "is dropped". After this diff such a post is refused with a way forward and nothing is stored, and a post while the relay is down returns `receiver_live: false` with a `creel: warning:` line. The orch's instructions do not describe either answer, and they are the very reader the board entry wants to report the result in its pane. Evidence: `contracts/orchrole/role-slack.md:7,16` against `internal/creelcli/slackbridge.go:121-124` and `internal/creelengine/verbs.go:193-196,212-214`.

Tool calls:
1. Searched for MachineBridge implementers and SLACK prose in contracts and plugins: the manifest held no implementer search and no stencil search.
2. Searched the docs for `receiver_live` and for the topics the help text removed: to check whether the removed help topics are still documented elsewhere.
3. Read the SLACK and button lines of `contracts/orchrole` and the bridge users in `cli_test.go` and `pool.go`: the stencil text and other bridge users were not in the manifest.
4. Read the hook and creelcli tests that post to SLACK and the head of `role-slack.md`: to confirm the stencil's wording and the existing posts to SLACK.
5. Read the bridge mentions in `creelengine/doc.go` and every `Refuse` call site: to confirm the package doc's bridge section and the shared-map cross-effect.
