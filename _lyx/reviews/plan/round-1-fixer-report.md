# Fixer report — plan round 1

## Changes

- **F1** (fixed as proposed): in `_lyx/plan/17-creel-read-sets.md`, the `TestDaemon_Protocol` sentence now replaces the row's last assertion with a discriminating one.
  After the ack, `programClaim("ly:BATTEN")` and alpha each post one message to alpha, and a `Request` with `All` set must hand out both, where plain `read` would hand out only the older sender's one.
  That keeps coverage of `All` crossing the protocol.
  `TestDaemon_Protocol` stays under `Edit:`.
- **F2** (fixed as proposed): the `Read_SetsAndInbox` sentence now also has the row read with `from` set to batten and `all` set while `b1` and an alpha message are still unread, and expect only `b1`.
- **F3** (fixed as proposed, with one wording correction): after the `read --wait` sentence, a new sentence says `Store.Read`'s doc comment names the three sets instead of saying `all` adds the read messages still retained.
  The grep sentence now lists the sites this card edits: `verbs.go`'s `Store.Read` comment, `protocol.go`'s `Request.All` comment, `cli.go`'s `readCmd`, `doc.go` and `server_test.go`'s `TestDaemon_Protocol` message.
  The proposed wording said the grep hits "only" those sites, but a repo-wide grep also hits `internal/reedcli/watchdog.go:190` and `cmd/testtiming/redundancy.go:106`, which use the phrase for something else.
  So the sentence limits the grep to `internal/creelengine` and `internal/creelcli`, where it holds.

## Disputed

None.

## Deferred

None.
