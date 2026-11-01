---
round: 1
focus:
  - "internal/heddleengine/shipped.go, override.go, render.go: do rendering, show and passages all read the shipped layer and the repository's heddle.json together for every target, shipped entries first as their own section, and does an override replace a shipped entry's text only when heddle.lym_override is on?"
  - "internal/heddlecli/verbs.go: does set/remove on a shipped slug refuse without --layer lym, does set --layer lym refuse when the setting is off, and does list mark each entry's layer and each overridden entry?"
  - "internal/shuttleengine/claudeengine/settings.go and shipped.json: does the batching sentence leave the appended system prompt and become the shipped layer's first entry aimed at claude-file, so a session never gets it twice, with the text unchanged?"
  - "Scope against the decision record's Added after Discussion entries: does the diff make changes none of them license, or omit one (the loomyard hub turning heddle.lym_override on, the heddle-orch-target target, the PATTERN and docs updates)?"
  - "internal/configreg, configcli, cmd/lym/configstrictness_test.go: is heddle.lym_override registered as a hub setting, default off, with the config-layers pattern consistent with it?"
  - "Failure paths in override storage in the Bolt checkout beside heddle.json: a missing, malformed or stale override (slug no longer shipped), and slug collisions between the shipped and repository layers."
  - "Test economy in heddleengine and heddlecli tests: duplicated coverage, and any new behavior without a test."
---

Seed focus from the decision record's goal and the diff's file list; no prior round exists.
