# creel: tell a sender when a post to the Slack relay cannot reach Slack

`lym creel post ly:SLACK "<text>"` answered `{"ok":true}` while Slack was disconnected, so an orch believed its result was on Slack when no channel had received it.
A post to the hub's Slack relay program now says what happened.

## What changes

- A post or `post --reply` from a strand or EXTERNAL to `<shortname>:SLACK` is refused, with nothing stored, when the relay would drop it unposted:
  the sender is no orch strand, its orch belongs to neither the hub nor a beam, or its slack button is off.
  Each refusal names a way forward: press the button on the status bar, ask the hub's orch or a beam's orch to post it, or report the text in the pane.
- When the sender is allowed but the relay is not running, the post is stored and answers `receiver_live: false` and `delivered: false`.
  A `creel: warning:` line names `lym slack start` and says the relay posts the waiting mail only if it starts within a day.
- When the relay runs, the post answers `receiver_live: true` and `delivered: true`.
- A read that fails while checking (geometry, beam listing, forwarding record, relay lock) refuses the post with a way forward of `lym slack status`.
- `role-slack.md` now tells an orch that a post with the button off is refused and one with the relay down answers `receiver_live` false, and to report the text in its pane on either.

## Design decisions

- The check lives behind a new `MachineBridge.RelayPostReady(sender)` method in `creelengine`, implemented by `slackBridge` in `creelcli`.
  The engine cannot read the Slack files itself, and the bridge already reads them afresh per call.
- The engine asks only when the receiver is this hub's relay program, the sender is not a program, and a bridge is told.
  A program's post, a post to a beam's relay program and a daemon without the bridge behave as before.
- `delivered` is a new response field, set only for the relay program.
  It carries the same answer as `receiver_live` for that receiver, so a caller can read it without knowing that `receiver_live` means "relay runs" here.
- `Store.Reply` became `Store.ReplyWith` and returns a `PostResult`, so a reply to relay mail runs the same check and carries the same fields and warning.
  The reply's check reads the store under its lock, then runs the bridge with no lock held, since the bridge reads files.
- An orch thread counts as the orch of its worktree for the button check.
- `lym creel post --help` lost a few sentences (the idle-wake clause, the reply-mark note, the resolved-receiver note) to make room under the help text's size cap.

## To check by hand

- Run `lym creel post ly:SLACK "<text>"` against a hub with the relay stopped and with an orch whose button is off, and read the answer, stderr and the refusal.
  This was not run against a live relay.
- Finding `relay-receiver-guard-unpinned` (design, medium): confirm the engine's receiver guard in `internal/creelengine/verbs.go` (`slackRelayCheck`)
  is pinned by a test for each receiver it skips, a strand, a beam's relay program and a store without a bridge.
  `TestStore_PostToTheSlackRelay` in `internal/creelengine/machinemail_test.go` has these rows.
- Finding `reply-to-slack-mail-skips-relay-check` (scope, medium): confirm a reply to relay mail, including a reply to a chase, runs the check in `relayReplyCheck` in `internal/creelengine/verbs.go`.
