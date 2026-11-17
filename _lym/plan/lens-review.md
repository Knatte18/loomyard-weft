# Lens review — slack-machine-bridge plan

Every fork returned a report; none is missing.
Findings that several lenses raised are merged and listed once, under the first lens that raised them.

- guard-reach + way-forward, BLOCKING (card 10): a settle lost to a dropped connection let the next take post the message again — fixed as proposed: `send` settles a taken id `Accepted` already holds with its recorded timestamp instead of posting it, with a row for the within-run case.
- guard-reach + way-forward, BLOCKING (card 5): `Reply` to machine mail skipped D2's orch check and `SendReady` — fixed as proposed: the reply branch runs the orch-sender check and `bridge.SendReady` before journaling, with rows for a button-off orch and a forwarded non-orch strand.
- guard-reach + coverage + row-validity + way-forward, BLOCKING (cards 8, 9): `BotUsersNamed` with an empty name was undefined, so `approve <m> <app id>` matched nothing — fixed as proposed: card 8 states that an empty name keeps every live bot and adds a `TestAPI` row; card 9's no-match refusal now names `status` as the way forward.
- row-validity, BLOCKING (cards 10, 11): unexported sender, poll and reader were unreachable from the external tests — fixed as proposed: `Create` handles for the `export_test.go` aliases and wrappers (`BridgeSender`, `BridgePoll`, `BridgeReader`, with `Send`, `SettleAccepted`, `Run`, `Wake`, `Read`, `Resync`).
- row-validity, MEDIUM (card 6): the `TestCreelCLI` rig's daemon had no `Options.Bridge` — fixed as proposed: `rig` gains a stubbed `slackBridge` field that `rig.startDaemon` passes; the `serveDaemon` line itself stays reached by review, like its existing `Hubs` line.
- row-validity + forced-edits, MEDIUM (cards 9, 10): `newRelayRun`'s own `Hub` literal left `MachineNames` nil, and `TestRunRelay` was unlisted — fixed as proposed: card 9 lists `newRelayRun` and states that a nil `MachineNames` is a wiring fault; card 10 lists `TestRunRelay` and `newRelayRun`, with an empty take queue and the bridge waits told apart.
- row-validity, MEDIUM (card 3): the start-scan row could not fail on a misrouted message — fixed as proposed: the row advances the clock past the hour and asserts no `outbound-returned` or `remote-delivered` record.
- row-validity + forced-edits, LOW (card 9): the reworded `status` answer broke existing exact rows — fixed as proposed: the Intent rewords those rows and words the unknown-app-id answer.
- row-validity, LOW (card 10): `already_in_channel` cannot come from `fakeSlack` — fixed differently: the proposal was to cut or re-aim the row. Instead the row reads "an invite that then stands clears the hold", because `API.Invite` maps `already_in_channel` to nil (api.go:320).
- row-validity, LOW (card 3): the three `SendReady` refusal rows repeated one branch — fixed as proposed: they are collapsed into one row, and the refusal wording is left to `TestSlackBridge`.
- row-validity, LOW (card 3): `env.start` was missing from Uses — fixed as proposed.
- row-validity, LOW (card 4): no row reached the `machine-report` dispatch — fixed as proposed: card 6's `TestCreelCLI` row reports a hold and reads `machine-held` from the log.
- row-validity, LOW (card 11): the holder row's "rereads no orch channel" could not fail there — fixed as proposed: the holder row asserts that no `resync` hint is written, and the read side rests on the `bridge` machine row.
- coverage, MEDIUM (cards 10, 11): the D10 wait criterion was only partly reached — fixed as proposed: card 11 adds rows for re-read doubling, the reset after a delivery, and the hint wake with its `wakeFloor`.
- coverage, MEDIUM (card 11): `handle` branches 1, 3, 4, 10 and 13 had no rows — fixed as proposed: rows added for the skips and for a lost-connection deliver.
- coverage, MEDIUM (card 11): an `approve` naming this relay's own machine in a bridge channel was not checked to be left alone — fixed as proposed: row added.
- coverage, MEDIUM (card 9): `runPeerCommand`'s refusal branches had no rows — fixed as proposed: rows added for a bad name, an unknown app id, revoke with no pin, and a Slack refusal through `retryOrGiveUp`.
- coverage, MEDIUM (cards 3, 5): `WaitReply` re-entry was untested — fixed as proposed: card 3 adds a register and refuse row, and card 5 a row where a reply that already arrived answers at once.
- coverage, MEDIUM (card 4): no-op branches and the one-second floor had no rows — fixed as proposed: rows added for a report on a settled message, a settle-returned no-op, recorded waits of at least a second, and an unknown-id settle refusal.
- coverage, MEDIUM (card 5): two `DeliverMachine` refusals had no rows — fixed as proposed: rows added for a bad machine part, a program name part and an over-cap line.
- coverage, LOW (card 3): the `Names` error branch was unreachable — fixed as proposed: `creelfake.Bridge.FailNames` and a row.
- coverage, LOW (card 10): the record limits the retried create, not only the listing — fixed as proposed: create and listing both run at most once per ceiling during a hold.
- coverage, LOW (card 10): sending branches had no rows — fixed as proposed: rows for `channel_not_found`, transport and rate-limit failures, and dropping a `Refusals` entry.
- coverage, LOW (card 11): the resync's former-name and timing cases were missing — fixed as proposed: rows added.
- coverage, LOW (card 11): the resync operand read "live hub" where the record says "a hub of this machine" — fixed as proposed: the Intent names the record's operand and reads it through `LiveHubs`, the relay's one listing of the machine's hubs (`internal/slackcli/wiring.go:146-166`).
- coverage, LOW (card 10): no unknown-key row for the post and notice records — fixed as proposed: row added.
- coverage, LOW (card 12): refusal branches had no rows — fixed as proposed: rows for a refused archive and its retry, and for a refused invite.
- coverage, LOW (card 2): the old-binary criteria rest on an argument with no row — disputed, every fix worse. This binary knows the `machine-*` events, so no row in this tree can fold them as unknown. The path an older binary runs, the default branches of `apply` and `applyCarried`, is pinned by `TestJournal_Fold`'s row "an unknown key and an unknown event name are ignored" (`internal/creelengine/journal_test.go:315`), which folds an unknown event and an unknown carried `of` to nothing.
- way-forward, MEDIUM (card 3): the over-cap refusal pointed at a file path the other machine cannot read — fixed as proposed: the refusal has its own wording, "shorten or split".
- way-forward, MEDIUM (card 4): the hour's return notice offered only "post again" — fixed as proposed: the machine notice names the way forward for each hold reason (`approve <machine>`, the relay notice, the button).
- way-forward + guard-reach, MEDIUM (card 10): send-side marks were never cleared after a return at the hour — fixed as proposed: `send` clears a mark whose pair and kind no taken message is held for, with a second-episode row.
- way-forward + guard-reach, MEDIUM (card 11): deliver refusals other than reply-to had no outcome — fixed as proposed: every answered refusal is skipped with a notice quoting creel's reason, with a row.
- way-forward, MEDIUM (card 10): the `bridge-name-taken` notice named no way forward — fixed as proposed: the notice names the rename.
- way-forward, LOW (card 10): notice destinations and a refused operator invite were unstated — fixed as proposed: notices go to O's channel and are mailed to O; a refused operator invite sends one notice and is retried by `inviteOperators`, with a row.
- way-forward, LOW (card 11): the wrong-app notice named no next step — fixed as proposed: it names comparing with `status` and `approve <machine> <app id>`.
- way-forward, LOW (cards 3, 1): the nil-bridge guard's way forward could not succeed, and the name-part refusal had none — fixed as proposed: the guard now says to restart the daemon, and the parser names `<machine>::<shortname>:orch`.
- way-forward + guard-reach, LOW (cards 4, 10): a refused report or settle for an unknown id had no handling — fixed as proposed: both refuse naming the id; the relay skips a refused report and drops an `Accepted` entry on an answered settle refusal.
- guard-reach, MEDIUM (card 11): read-side marks were never cleared — fixed as proposed: a mark ends when a read moves the cursor past a message not in that state, with a new-episode row after `revoke`.
- guard-reach, LOW (card 11): a bridge thread hint fell through to `readsForHint` and was dropped — fixed as proposed: `takeHints` diverts a thread hint whose channel the bridge map holds, `readsForHint` is listed under Edit, and a row covers the wake.
- guard-reach, LOW (card 12): a refused bridge archive could never be retried — fixed as proposed: `retire` also visits an orch that still holds open bridge entries, with a row.
- guard-reach, LOW (card 11): the creator's app id had no source — fixed as proposed: card 8 adds `API.UserAppID` (users.info `api_app_id`), and card 11 uses it.
- forced-edits, MEDIUM (card 6): the creel CLI's JSON writers dropped machine-mail fields — fixed as proposed: `messageFields` and `logEntryFields` are added under Edit, with rows and doc text.
- forced-edits, LOW (card 3): the `PostResult`, `PostTags`, `postCmd` and `forwardCmd` comments and help were left stale — fixed as proposed: added under Edit, with `internal/creelcli/doc.go` in Uses.
- forced-edits, LOW (card 1): the package-comment edit had no target — fixed as proposed: `internal/agentname/agentname.go` is added under Extends.
- forced-edits, LOW (card 8): `ChannelInfo` was under Edit though its fields do not change — fixed as proposed: moved to Uses.
