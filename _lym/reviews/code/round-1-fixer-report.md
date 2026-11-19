---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: The review put the check inside Store.Reply's locked closure; it now runs in relayReplyCheck before the store lock, since the bridge reads files and a hung read would stall every verb. The result, VerbReply response and CLI --reply branch carry live, delivered and the warning as proposed.
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
---

## Changes

- F1: `Store.ReplyWith` (new, returns `PostResult`) runs `relayReplyCheck` before taking the store lock, resolving a chase to the chased sender.
  A refusal returns before anything is journaled.
  `server.go` fills the `VerbReply` response from it, and `cli.go`'s `--reply` branch prints `receiver_live`, `delivered` and the warning line.
  `Reply` moved to `export_test.go` as the test wrapper, beside `Post`.
  Rows for a live relay, a down relay and a refused reply are in `TestStore_PostToTheSlackRelay`.
- F2: rows post to a strand (`alpha`) and to a beam's program (`ly:zz:SLACK`) with the bridge refusing and the relay down; the journal filter follows the row's receiver and sender.
- F3: `Delivered *bool` on `PostResult` and `Response`, carried through the plain and waiting post responses and the reply, printed as `delivered` by the CLI; pinned in the live and down rows.
- F4: `RelayPostReady` wraps each read failure in `relayUnreadable`, which ends in a way forward naming `lym slack status`; `TestSlackBridge_RelayPostReadyRefusesWithAWayForwardWhenAReadFails` pins it.
- F5: `relayDownWarning` says the relay posts the waiting mail only if it starts within a day and drops older mail; the test constant matches.
- F6: an EXTERNAL row in `TestStore_PostToTheSlackRelay` and in `TestSlackBridge_RelayPostReady`.
- F7: `contracts/orchrole/role-slack.md` names the refusal and the relay-down answer.
- F8: `_lym/landing/summary.md` moves the slack turn hook out of the program senders and records `delivered` and the reply check.
  The file is not committed, since it sits under `_lym`.
- Self-check follow-ups: the `creelengine` package doc, `MachineBridge.RelayPostReady`, the `Response` and `PostResult` comments and the `post` help text now cover replies and `delivered`.

A gate then flagged comment lines of 120 characters or more in `creelcli/doc.go`, `creelcli/slackbridge.go`, `creelengine/protocol.go` and `creelengine/verbs.go`.
Each is now split into one sentence per line, with no change of wording.

A verify run then failed `TestCLIForm_EveryCommand`: the `post` help text, grown by the `delivered` mention, was 2004 bytes against the 2000-byte cap.
The help paragraph on the relay is now shorter, and it no longer says that any other sender is refused; the gate for `cmd/lym` and `creelcli` passes.

Commits: `729cce81` (F4, F6 bridge), `a5279497` (F7), `9d549b5f` (F1, F2, F3, F5, F6 engine), a commit for the self-check changes, and a last one for the comment line breaks.
F1, F3 and the shared test table are intertwined, so they share a commit.
The gate for `creelengine`, `creelcli` and `creelfake` passes.

## Self-check

- fix-code-correctness, MEDIUM: the bridge ran under the store mutex in `ReplyWith` — fixed differently with F1 (check before the lock).
- fix-consistency, MEDIUM: `creelengine/doc.go` still described a post only — fixed.
- fix-consistency, LOW: `MachineBridge.RelayPostReady` doc — fixed.
- fix-consistency, LOW: `post` help text lacks `delivered` — fixed within the size cap.
- fix-consistency, LOW: `Response` comments cover only a post — fixed.
- fix-consistency, LOW: `PostResult` comments cover only a post — fixed.
- fix-proposal, MEDIUM: the same lock finding, merged with fix-code-correctness.
- fix-proposal, LOW: `delivered: true` overstates a live relay — fixed by documenting the field as whether the relay was running to take the mail.
- fix-proposal, LOW: `Reply` as a production wrapper used only by tests — fixed by moving it to `export_test.go`, as `Post` already is.
- fix-affected-set, LOW: `Store.Forward` to the relay program is unchecked — deferred, reason below.
- fix-affected-set, LOW: `post` help text — merged with the fix-consistency finding.
- fix-code-correctness note: no test asserts `delivered` in the `Response` JSON or the CLI envelope; below the rubric's LOW, so not acted on.

## Disputed

None.

## Deferred

- Self-check finding on `Store.Forward`: forwarding mail to `<shortname>:SLACK` skips the relay check.
  Covering it extends the board entry from post and reply to forward, and a forward answers no liveness fields, so a down relay could not warn.
  Whether the entry reaches forward is a scope call for the parent or operator, so it is left undone.

## Rulings

None.
