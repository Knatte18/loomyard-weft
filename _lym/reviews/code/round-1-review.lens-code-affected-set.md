```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. I checked everything outside the touched lines that reads, calls or asserts what the diff changed.

**What each change reaches, and whether it is covered**

1. **`botUser` became `botIdentity`, and the keeper's `botUserID` field became `bot Identity`.**
   - Production callers: `keeper.go:202` `adopt`, `bridgechannel.go:122` `adoptBridge`, `bridgein.go:102` `load`, and `bridgein.go:685` `resync`, which feeds `resyncChannel`/`authored`. All are updated, and grep finds no other reference to `botUser`/`botUserID`.
   - Test fake: `inbound_test.go:646-650` returns `UserID: "UBOT", BotID: "B1"`, which matches the new rows.
   - Production client: `api.go:74-81` already decodes `bot_id` into `Identity`, and `api_test.go:178` asserts it. Covered.
2. **`isOwnPost` replaces the user-id comparison in `handle` and `authored`.**
   - Other read paths that judge authorship: `isOperator` (`bridgein.go:254`) requires `BotID == ""`, so the bot's own posts are already excluded there. `readThread` (`bridgein.go:631-656`) only acts on operator replies.
   - Creator checks at `keeper.go:206`, `bridgechannel.go:126` and `bridgein.go:734` compare a channel creator, which is always a user id, so `bot.UserID` is the right field there.
   - Covered.
3. **Relay command syntax is now `lym &lt;command&gt;`** (`inbound.handle`, `relayCommandWords`, `parsePeerCommand`). It reaches:
   - Readers: bridge `operator` (`bridgein.go`), which shares `parsePeerCommand`.
   - Tests: `inbound_test.go` and `bridgein_test.go` rows are updated, and grep finds no bare `Text: "approve…"`/`"status"` left in the slackengine tests.
   - Docs and contracts: `docs/slack-app.md`, `contracts/orchrole/role-slack.md` and `internal/slackengine/doc.go` are updated.
   - `docs/overview.md:947` is **not** updated (finding F1).
   - `internal/slackcli/cli.go:148-152` describes the CLI verbs `lym slack away`/`back`/`status`, not Slack text, so it is unaffected.
4. **Way-forward and notice wording moves to `lym approve`/`lym status`.** It reaches `creelengine/machinemail.go:237` and its test `machinemail_test.go:165`, `bridgeout.go:216` with `bridgeout_test.go`, and `bridgein.go` notices with `bridgein_test.go`. All are updated. The `peer-unapproved` reason keys are unchanged. Covered.
5. **Rule documents.** No PATTERN change is declared. `PATTERN-agent-addressing` does not describe the command syntax. Both fix commits carry a `Root-Cause:` trailer, and none carries a `Pattern-Departure:` or `Loosened-Check:` trailer. Covered.

**Findings**

- **F1 — MEDIUM — consistency — `docs/overview.md:947`**
  - Summary: the overview's slack entry still says the relay "answers `away`, `back` and `status` in the channel". The diff makes those bare words ordinary messages that are forwarded to the orch, and only `lym away`, `lym back` and `lym status` are answered. The overview now states the opposite of the code and does not mention the `lym` prefix.
  - Evidence: `docs/overview.md:947` against `internal/slackengine/inbound.go:684-693` (`relayCommandWords`, which requires the prefix).
  - Fix: name `lym away`, `lym back` and `lym status`, and add `lym approve`/`lym revoke`, as `docs/slack-app.md`'s "Relay commands" section does.

**Tool calls**

1. Grep over the whole repo for bare "type/answers approve|revoke|status|away|back" and for docs mentioning the Slack relay. My context only covered the slack, creel, docs and contracts paths.
2. Grep of `docs/overview.md`, `orch-role.md`, `role-autonomy.md` and the non-slack tests for the old forms. These are docs that describe the relay and that the manifest had not read.
3. Grep for `peer-unapproved`, `botUser` and bot-user-id wording, plus the `slackcli` help text. This looked for other readers of the changed messages and identity.
4. Read `internal/slackcli/cli.go:140-155` to judge whether the presence help text describes Slack commands. My context held only the grep line.
