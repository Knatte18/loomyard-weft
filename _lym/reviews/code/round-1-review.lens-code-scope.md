```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to the board entry's goal points, which serve as the decisions.

**Decisions** (the record has only `## Goal`, with no `Added after Discussion` entries, and its other sections are empty):

- **G1. Skip own posts by bot id.** Skip a message whose `bot_id` is this machine's bot (from `auth.test`) and keep the `user` check.
  - Built by C1 and C2.
- **G2. Test the Slack post shape.** A test feeds a post shaped as Slack returns a `username`-override post: `subtype: bot_message`, `bot_id`, no `user`.
  - Built by C3.
- **G3. Prefix every relay command.** Every relay command takes one prefix (`lym approve neve`, `lym status`, `lym away`), and every other message is forwarded unchanged.
  - Built by C4.
- **G4. Name the new form everywhere.** The relay's answers, `docs/slack-app.md` and `contracts/orchrole/role-slack.md` name the new form.
  - Built by C5 and C6.

**Changes:**

- **C1 → G1.** `channelKeeper.botIdentity` keeps the whole `auth.test` identity and replaces `botUser`.
  - It is at `internal/slackengine/keeper.go:591-607`.
  - Its callers change only because the type changed, so they are forced edits: `keeper.go:202,206` `adopt`, `bridgechannel.go:122,126` `adoptBridge`, `bridgein.go:685,734` `resync` and `resyncChannel`.
- **C2 → G1.** `isOwnPost` compares the user id or the bot id (`bridgein.go:106-110`).
  - `handle` uses it at `bridgein.go:267`, which is the bridge poll's own-post skip.
  - `authored` also uses it at `bridgein.go:778`. The board entry names only the poll, but `authored` is the same own-post identification with the same defect. The commit message's `Root-Cause` names it and the summary states it. I read it as in G1's scope, not unrequested.
- **C3 → G2.** Two test rows use a `bot_message` with `B1` and no user:
  - `bridgein_test.go:279-281` (the skip);
  - `bridgein_test.go:678-680` (the resync adoption, traced through C2).
- **C4 → G3.** The prefix and the forwarding of everything else:
  - `relayCommandPrefix` and `relayCommandWords` (`peercommand.go:16-33`);
  - `parsePeerCommand` (`peercommand.go:45-63`);
  - `inbound.handle` (`inbound.go:684-693`) and the command constants (`inbound.go:41-44`);
  - test rows `inbound_test.go:1526-1533`, where a bare `status`, `approve it` and `lym board list` are forwarded.
- **C5 → G4 (the relay's answers).** These strings now name the `lym` form:
  - `bridgein.go` wrong-app, unpinned and unapproved-bot notices;
  - `bridgeout.go:216` hold line;
  - `peercommand.go` answers.
  - Creel's way forward at `machinemail.go:237` is not strictly a relay answer, but the prefix change would leave it stale, so it is a forced edit.
- **C6 → G4.** `docs/slack-app.md` pairing steps and the new `Relay commands` section, and `contracts/orchrole/role-slack.md:15-16`.
- **Forced edits.**
  - The `doc.go:35-37` package doc is a rule text the change makes stale.
  - Comment wording in `bridgein.go:491`, `inbound.go:672-673,723-724,736` and `peercommand.go:1-2,32,45` is stale otherwise.
  - Test literals updated to the prefixed form in `bridgein_test.go`, `bridgeout_test.go`, `inbound_test.go` and `machinemail_test.go`.
  - Commit 45796424e splits comment lines, which the comment lint forces.
- **Untraced changes:** none.

**Findings:** none. Every change traces to G1–G4 or is an edit a traced change forces, and every goal point is built.

Tool calls: none. Everything this lens needed was already in my inherited context.
