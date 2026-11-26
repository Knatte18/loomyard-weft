```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens over the diff's changed functions and branches.

**Functions and branches run**

- **`isOwnPost` (`bridgein.go:108`):** correct in every case run.
  - With bot `{UBOT, B1}`, a message `{Subtype: bot_message, BotID: B1, User: ""}` gives true.
  - `{User: UBOT}` gives true.
  - `{User: U9}` and `{BotID: B9}` from the peer's bot give false.
  - An empty `Identity{}` against a message with empty `User` and `BotID` gives false, because the `!= ""` guards hold.
  - An identity with only one of the two ids set matches on that id alone.
- **`bridgeReader.handle` (`bridgein.go:267`):** correct.
  - Run on the incident's post: this machine's username-override envelope with `from=home`, carrying `BotID B1` and no user. It is now skipped before `handleEnvelope`, so the from-clash notice no longer fires.
  - The operator's message (`User = operatorID`, no `BotID`) still reaches `isOperator`.
- **`bridgeReader.load` (`bridgein.go:102`):** correct.
  - When `botIdentity` fails, the error returns before any channel is read, the same as with `botUser` before the change.
- **`bridgeReader.resync` and `resyncChannel` (`bridgein.go:685`, `:734`):** correct.
  - `creatorTrusted` still compares the channel creator, a user id, with `bot.UserID`.
  - An `auth.test` failure returns before any channel is judged.
- **`bridgeReader.authored` (`bridgein.go:778`):** correct.
  - The no-user envelope with `BotID B1` counts as authored, so a channel holding only such posts is adopted.
  - A peer-bot message still goes through the `appOf` pin check.
- **`channelKeeper.botIdentity` (`keeper.go:592`):** correct.
  - The first call caches `{UBOT, B1}`; later calls return it without calling `auth.test`.
  - When `auth.test` returns an empty `UserID`, nothing is cached, so every call re-reads it. That matches the old behaviour.
  - An `auth.test` error is returned wrapped, and the cache stays empty.
  - The error text still reads "read the bot's user id". This is cosmetic and not misleading enough to file.
- **`channelKeeper.adopt` (`keeper.go:206`) and `adoptBridge` (`bridgechannel.go:126`):** correct.
  - Both compare `Creator` with `bot.UserID`, unchanged in meaning.
- **`relayCommandWords` (`peercommand.go:601`):** correct.
  - `""` and `"lym"` give false.
  - `"LYM Back"` gives `["Back"]`.
  - `"lym\napprove neve"` gives `["approve", "neve"]`, because `strings.Fields` splits on any whitespace.
  - `"lymx status"` gives false.
- **`parsePeerCommand` (`peercommand.go:622`):** correct, and as the board entry asks, a malformed command is forwarded to the orch unchanged.
  - `"lym approve neve"`, `"lym approve neve A0X"` and `"lym revoke neve"` parse as peer commands.
  - `"lym approve"`, `"lym approve a b c"`, `"approve neve"` and `"lym revoke a b"` give false and are forwarded.
- **`inbound.handle` presence branch (`inbound.go:684`):** correct.
  - `"lym away"`, `"lym back"` and `"lym status"` dispatch to their handlers.
  - `"lym away now"` and `"lym board list"` fall through to `parsePeerCommand`, which gives false, and are forwarded.
  - A bare `"status"` is forwarded.
  - With the button off, `"lym away"` is still answered, as before the change.
- **Bridge `operator` (`bridgein.go:494`):** correct.
  - `"lym approve osl"` naming the peer runs and is answered.
  - A bare `"approve osl"` gets the "not delivered" answer.
  - `"lym away"` in a bridge channel is not a peer command, so it gets the same answer.
- **Rewritten format strings:** correct. In `reportUnapprovedBot`, the wrong-app notice in `handleEnvelope`, the unpinned-hold notice, `sendOne`'s hold line, the `handlePeerCommand` refusal, `approvePeerByName` and `approvePeerByAppID`, every placeholder count matches its argument count.
- **`machineHoldWayForward` (`machinemail.go:237`):** correct; `machine` is substituted into the backticked `lym approve`.

**Findings**

None. Every changed function and branch gives the right result on the inputs above, and every failure path returns its error with no state left half-written.

Tool calls: none. My inherited context held every definition and call site this lens needed.
