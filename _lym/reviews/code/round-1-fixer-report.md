---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
---

## Changes

- F1 (`97b14030c`): the slack entry of `docs/overview.md` now says the relay answers `lym away`, `lym back`, `lym status`, `lym approve` and `lym revoke` in the channel.
- F2 (`549ec28fb`): `TestBridgeReceive`'s resync table in `internal/slackengine/bridgein_test.go` gained the row `MadeByThisMachinesBotPostingWithItsUserIDAndNoBotID`.
- F3 (`9cb04106a`): the relay-command table of `TestInboundRead` in `internal/slackengine/inbound_test.go` gained the row `APresenceCommandWithMoreWordsIsForwarded`.

`lym gate test ./internal/slackengine` passed with all three changes in the tree.
The two new rows were not run against a mutant; that each fails with its defect in place was traced by hand.

## Self-check

No fork returned a finding.

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

The fix-proposal lens also noted, outside its lens and not as a finding, that `docs/overview.md` says the operator's message in Slack sets the presence to `slack` while `lym status` leaves it unchanged.
This round did not change that sentence.

## Disputed

None.

## Deferred

None.

## Rulings

None.
