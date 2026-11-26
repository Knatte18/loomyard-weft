---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:947
    summary: The overview's slack entry still says the relay answers bare away, back and status in the channel, which the diff now forwards to the orch; only the lym-prefixed forms are answered.
    fix: "In docs/overview.md, in the slack paragraph's sentence that opens \"The relay forwards the operator's messages in an orch's channel\", replace \"answers `away`, `back` and `status` in the channel\" with \"answers the relay commands `lym away`, `lym back`, `lym status`, `lym approve` and `lym revoke` in the channel\"."
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/slackengine/bridgein_test.go:678
    summary: No test decides isOwnPost's kept user-id clause; every own-post fixture also carries bot id B1, so the mutation survivor at bridgein.go:109 passes every test.
    fix: "In TestBridgeReceive's resync table, beside the row MadeByThisMachinesBotPostingUnderAnOrchsNameWithNoUser, add a row MadeByThisMachinesBotPostingWithItsUserIDAndNoBotID with creator UBOT and one envelope {homeOrch, peerOrch, \"UBOT\", \"\"}, expecting the same adoption (primeOrch, peerOrch)."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/slackengine/inbound_test.go:1532
    summary: No row pins that a presence command word with extra words, such as lym away now, is forwarded rather than run; removing the len(words) == 1 condition at inbound.go:313 passes every test.
    fix: "In TestInboundRead's OperatorMessageSetsThePresenceToSlackBeforeItIsForwardedAndRelayCommandsAreNotForwarded table, after the row ThePrefixBeforeAnUnknownCommandIsForwarded, add {\"APresenceCommandWithMoreWordsIsForwarded\", \"lym back now\", slackengine.PresenceSlack, slackengine.PresenceSlack, []mail{{primeOrch, \"lym back now\"}}}."
---

### [MEDIUM] The overview still names the bare relay commands

**Location:** `docs/overview.md:947`
**Issue:** The slack paragraph says the relay "answers `away`, `back` and `status` in the channel".
After the diff, `inbound.handle` answers only `lym away`, `lym back` and `lym status`, and forwards a bare `status` to the orch (`internal/slackengine/inbound.go` `handle`, `peercommand.go` `relayCommandWords`).
The board entry asks that the docs name the new form; this doc now states the opposite of the code.

### [LOW] isOwnPost's user-id clause is untested

**Location:** `internal/slackengine/bridgein_test.go:678`
**Issue:** The board entry asks to keep the `user` check beside the new `bot_id` check.
Every own-post fixture carries `BotID: "B1"` (`bridgein_test.go:278`, `:676`, `:679`), so the mutation report's survivor at `bridgein.go:109`, a negated comparison in `isOwnPost`, passes every test.
In `handle`, a post with no bot id is skipped by a later check anyway, so the resync's `authored` is the path where a row can decide the user clause.

### [LOW] A presence word with extra words has no forwarded row

**Location:** `internal/slackengine/inbound_test.go:1532`
**Issue:** `inbound.handle` runs a presence command only when `relayCommandWords` returns exactly one word (`inbound.go:313`).
The board entry asks that every message that is not a relay command be forwarded unchanged.
No row feeds a text such as `lym back now`, so dropping the one-word condition would run `back` and swallow the message without a test failing.

## Rejected

- code-tests' finding on `keeper.go:604` (`botIdentity` cache write untested): the mutant drops a pure cache of `auth.test`'s answer, which the rubric names as a survivor no test need kill; behaviour is the same with or without the cache.
- code-tests' rows for a bare `lym` and a four-word `lym approve`: removing the two-field check in `relayCommandWords` changes no outcome for a bare `lym`, and the four-word refusal in `parsePeerCommand` predates the diff, so neither is a behaviour the diff adds; the kept row is narrowed to F3.
- Handler check, not raised: `docs/slack-app.md` ends its new `Relay commands` section with "The manifest above stays as it is." The line now follows the commands, where it explains why a typed prefix needs no manifest change, so it still holds.
- Handler check, not raised: the change description's `two-way` door and `slack` blast radius. The diff changes no record or board format, no config meaning, deletes nothing and pushes nothing outward. The creel change is way-forward text for the slack relay's own hold, and the resync writes the bridge map in its existing shape.
