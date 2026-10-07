---
round: 1
focus:
  - "Fidelity: check each Decision D1-D6 and each Constraint in the decision record against the cards. Settle whether any is carried by no card, in particular the `docs/overview.md` and `shuttleengine` and `claudeengine` `doc.go` docs scope, the D5 log line, and the D4 whole-set validation."
  - "Verify coverage: compare the packages the cards target (`shuttleengine`, `claudeengine`, `configreg`, `webstercli`) with what the `## verify:` section runs, including the `-tags integration` and `-tags tmux` tests of those packages, and check the section's order against the plan template."
  - "Test economy: for the new tests in cards 1 and 3 (`TestLoadConfig_PromptCacheTTLKeys`, `TestPrepare_LogsResolvedPromptCacheTTL`, the added `TestPrepare_ThreadsSpecIntoLaunchCmds` and `TestPrepare_RejectedBeforeArtifacts` rows), settle whether an existing test already covers the behavior or whether a row extends an existing table where a new function was planned."
  - "Attack surface: card 3 adds a refusal guard that spans every role, and a pane-line env assignment that outranks operator settings. Settle whether its `**Intent:**` states what it can block or let through and what bounds it."
  - "Granularity: settle whether the card boundaries (config keys, registry entry, claudeengine resolution with its docs and fixtures) are one reviewable unit each, and whether any card bundles two independently reviewable units or splits one."
---

Seed focus only; every rubric check still applies in full.
