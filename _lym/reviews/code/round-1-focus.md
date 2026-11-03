---
round: 1
focus:
  - "internal/yamlengine/set.go and layers.go: does a whole-map pair at an open map P accept a flow mapping for `capture` and `compare`, and what happens on a malformed flow mapping, a non-mapping value, an empty `{}`, or a nested value?"
  - "internal/yamlengine/set.go: the shape check for a key inside a whole-map value; does a declared shape refuse an unreached key by name, and does a shape-less open map accept any key?"
  - "internal/yamlengine/layers.go: the change to the declared-open-map set; does it alter how existing P.<name> pairs, SetValues or SetSparse behave for keys that worked before?"
  - "internal/driftengine/config.go against the decision record Goal: is the key description the one change the Goal's alternative allows, and does it still match what --set now accepts?"
  - "doc.go files in configcli, configengine and yamlengine: do the doc comments follow the repository's comment conventions and match the new behavior?"
  - "Test economy in set_test.go, layers_test.go and driftengine/config_test.go: does each new test cover behavior no existing test covers, and is any new behavior left without a test?"
  - "Commit message claims against the diff: 'every declared open map P', 'replaces it whole', and 'refused naming it'."
---

Seed focus for round 1. The answer key is a decision record alone: the board entry under `## Goal` (issue #19) with no `Added after Discussion` entries.
