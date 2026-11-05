---
round: 2
exclude_lenses: []
focus:
  - "Raddle-File ordering and retry in decision-record.md D9 (steps 1 to 5) against D8's filing-time return to Raddle-Write: when the board re-check fails and a revised proposal is approved, does every output an earlier attempt wrote (raddle store, part records) get rewritten rather than skipped, and does every step have a stated done marker for a crash-resumed retry?"
  - "How a part record becomes the stored decision record that D3's Record-Write reads: boardengine.StoreDecisionRecord (internal/boardengine/decision.go) refuses an absent or done entry, so check that D9 names the board write that creates the entry, stores its record and sets its discussed flag, and that the raddle store reaches origin with that commit."
  - "When raddle_auto_approve is read, against D7's Bound: internal/boardcli/runsettings.go tells a running pair to change settings through lym config with --layer local, and internal/configcli/hubwide.go refuses only hub-wide writes to task sessions. Check that the mode is fixed once and that Raddle-File applies D6's checks in both modes."
  - "The cost design in D12 to D14 against the repository's existing pricing: internal/shuttleengine/claudeengine/pricing.go prices per message through shuttleengine.MessagePricer, and a Codex rollout's token_count events carry no cache-write field. Check that the decisions use the engine pricer or give a reason not to, and that no fact about Codex usage is left open."
---

No lens is excluded: each lens that ran in round 1 carried at least one MEDIUM or worse finding.
