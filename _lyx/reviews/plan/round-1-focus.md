---
round: 1
focus:
  - "Fidelity: settle whether card 10 carries the rework finding F1 and the parent decision 'Template review models per segment' completely: `review: opus[medium]`, `webster_review: sonnet[high]`, and every other per-segment key left empty."
  - "Fidelity: settle whether any part of card 10's scope (the `wiring_test.go` assertion, the grep for other pinned values) is licensed by F1 or the decision record, or goes beyond them."
  - "Test pins: check the card's claims about `config_test.go` (`templateConfig`, the `TestLoadConfig_Loads` seeded-file row) and `wiring_test.go` (`TestWire_ReviewKeysReachTheEnv`) against those files, and ask whether another test in the tree pins the old template `review` or an empty `webster_review`."
  - "Test economy: settle whether the new `wiring_test.go` assertion duplicates behavior the `config_test.go` template pin or an existing wiring test already covers."
  - "Verify coverage: settle whether the `## verify:` section runs the tests of `internal/loomengine` and `internal/loomcli` and keeps the order the plan template states."
  - "Edit-card form: settle whether Edit targets that are only file paths, with an Intent naming specific keys and tests, fit the member-glyph rule for these file types."
---

Round 1 seed.
The plan is one card, so granularity and card order have little to settle.
The answer key is the decision record plus the live rework round's `findings.md` (F1).
