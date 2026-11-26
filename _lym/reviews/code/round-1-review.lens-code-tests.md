```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, I looked for the test row that would fail if it broke.

**Per behaviour, the test that pins it**

1. `handle` skips this bot's own post when it has a bot id and no user id → `TestBridgeReceive/MessagesThatAreNotForCreel…/ThisBotsOwnPostUnderAnOrchsNameWithNoUser` (bridgein_test.go:281). It asserts no delivery, no post and that the cursor moves past.
2. `authored` (resync) counts this bot's own envelope when it has no user id → `TestBridgeReceive` resync row `MadeByThisMachinesBotPostingUnderAnOrchsNameWithNoUser` (bridgein_test.go:678).
3. `isOwnPost` matches by user id alone, the user check the board entry says to keep → **none**. Every own-post row carries both `UBOT` and `B1`. Mutation survivor at bridgein.go:109.
4. `isOwnPost` with an empty `UserID` or `BotID` in the identity → **none**. The fake `AuthTest` always returns both ids (inbound_test.go:650).
5. `botIdentity` reads `auth.test` once and keeps the result → **none**. Mutation survivor at keeper.go:604 (`k.bot = identity`).
6. `botIdentity` passes on an `auth.test` failure → `keeper_test.go:272` `AuthTestFails`.
7. Creator checks now use `bot.UserID` (`adopt`, `adoptBridge`, `resyncChannel`) → `TestChannelKeeper`, `TestOutboundRead`, `TestBridgeSend`, and the `TestBridgeReceive` resync rows `MadeByThisMachinesBot` / `MadeByAnotherCreator`.
8. `lym away`, `lym back`, `lym status`, matched without regard to case → `TestInboundRead` presence steps (inbound_test.go:1496–1499, including `" LYM Back "`) and the `BackCommand` / `StatusCommand` / `AwayCommand` rows.
9. A bare `status` and a plain `approve it` are forwarded → rows `ABareCommandWordIsForwarded` and `ApproveInPlainWordsIsForwarded` (inbound_test.go:1530–1531).
10. `lym` followed by an unknown verb is forwarded → row `ThePrefixBeforeAnUnknownCommandIsForwarded` (`lym board list`).
11. A presence command with extra words (`lym away now`) is forwarded; this depends on the `len(words) == 1` condition (inbound.go:313) → **none**.
12. `lym` alone, and a peer command with too many words (`lym approve neve A0X extra`), are forwarded → **none**.
13. `lym approve` / `lym revoke` parsing, including mixed case and an app id → the `TestInboundRead` peer-command rows (inbound_test.go:1848–1873).
14. A bare `approve osl` in a bridge channel is no longer a command → `TestBridgeReceive` row at bridgein_test.go:187 (replaces `just a line`).
15. Bridge `lym approve` / `lym revoke` naming this machine → bridgein_test.go:150. In a thread → bridgein_test.go:196.
16. Every changed answer, notice and way-forward text naming `lym approve` / `lym status`:
    - wrong app → bridgein_test.go:159
    - unpinned hold → :169
    - unapproved bot → :224
    - outbound hold → bridgeout_test.go:251/255
    - naming this machine → inbound_test.go:481/487
    - no bot or two bots → :456/462
    - app id unknown → :473
    - creel way forward → machinemail_test.go:165
17. Docs, contract and `doc.go` prose → exempt.

**Findings**

- **LOW · design** · `internal/slackengine/bridgein.go:109` (`isOwnPost`)
  - Summary: no test row decides the kept user-id clause or the empty-id guards, so the line-109 mutant survives.
  - Evidence: every own-post fixture carries `BotID: "B1"` (bridgein_test.go:278, 676), and the fake identity is fixed (inbound_test.go:650). In `handle` a post with no bot id is skipped anyway (bridgein.go:273), so only the resync path can decide the user clause.
  - Fix: add a resync row in `TestBridgeReceive` whose envelope is posted with user `UBOT` and no bot id, and that expects the channel to be adopted. It pins the user check the board entry says to keep.
- **LOW · design** · `internal/slackengine/keeper.go:604` (`botIdentity`)
  - Summary: the doc comment promises "read once and kept", but no test checks it, so dropping the cache write survives.
  - Evidence: keeper.go:591 and 604. A search for an `AuthTest` call count in the tests found none.
  - Fix: assert that `fakeSlack` logs one `AuthTest` call across two reads.
- **LOW · design** · `internal/slackengine/inbound.go:313` and `peercommand.go:601` (`relayCommandWords` and its callers)
  - Summary: no row covers a relay command with extra or missing words: `lym away now`, a bare `lym`, `lym approve neve A0X extra`. The board entry says these must be forwarded unchanged.
  - Evidence: no such text appears in inbound_test.go. The `OperatorMessageSets…RelayCommandsAreNotForwarded` table (inbound_test.go:1528–1533) has only one-word and unknown-verb rows.
  - Fix: add forwarded rows for `lym away now`, `lym` and a four-word `lym approve` to that table.

**Tool calls**

1. `sed` of bridgein.go:264–300, bridgein_test.go:262–320 and keeper.go:598–607. My context lacked the body of `handle` past the own-post check, the skipped-messages test's assertions, and the line the keeper.go:604 survivor points at.
2. `grep` of the tests for extra-word relay command rows and an `AuthTest` call count. My context did not show whether such rows exist.
