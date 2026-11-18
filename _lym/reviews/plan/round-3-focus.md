---
round: 3
focus:
  - "Card 5 (`_lym/plan/05-account-pick-at-making.md`): `battenCLI.wire` closures for `CreateWorktree` call a `pickAccount` seam; settle whether every test that builds `&battenCLI{...}` and calls `wire` (`wireForHub` in `lifecycle_integration_test.go`, the beamfork integration test, the `CreateWorktree` rows of `wire_seams_test.go`) reaches a non-nil seam, and whether the card lists each builder it needs changed."
  - "Card 14 (`_lym/plan/14-in-run-relaunch.md`) and card 16: `loadSkillsThenPrompt` runs `relaunchUnderAccount` on the start path, and `launch` reads a nil error from `loadSkillsThenPrompt` as a started run; settle what the cards say is returned when the relaunch ends the run died, timed out, api-error or done, and which planned row reaches a failing relaunch through `Start`."
  - "Cards 14 and 16: every doc file listed under `Extends` must have no existing sentence falsified by the card's new behaviour; settle this for `internal/shuttleengine/doc.go` and for any other doc, spec or PATTERN file those cards list."
  - "Any text round 2's fixes added or moved in cards 5, 14 and 16: check card order, the attack-surface bound of any new guard or hold, and reachability of any prescribed refusal by a named input and test."
---

Round 2's three BLOCKING findings were fixes to cards 5, 14 and 16; the entries above ask the reviewer to settle each question on the card text as it now stands.
