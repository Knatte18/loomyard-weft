---
round: 1
focus:
  - "Fidelity, cards 2 and 3 against Decisions 5, 6, 7, 10, 11 and 12: does each card carry every bound the decision record states (one retry, the retry turn classified like the first, timeout and unverified never retried, the events offset past the retry's turn end, restart never retries twice, old `reload_step` read as the pointer step)?"
  - "Fidelity, Decision 14 and the Scope line for the two loomcli stubs: is each stub's change confined to its skill-load answer and comment, and does card 2's `Edit` set reach everything Decision 14 names without adding scope the answer key does not license?"
  - "Design, sequencing across cards 2, 3 and 4: card 4 claims no caller of `Runner.LoadSkill`, `Runner.SkillUnknown`, `SkillLoadSequence` or `SkillUnknown` remains after cards 2 and 3, and its `Verify:` greps for it. Do cards 2 and 3 together retarget every production and test implementer or caller of those names, so the tree builds and tests pass after each card, not only after card 4?"
  - "Verify coverage: does the `## verify:` package list run the tests of every package any card targets, including `internal/shuttleengine/claudeengine` and `internal/orchcli`, under every tag the cards' tests carry (the `tmux` stubs in `internal/loomcli`, untagged unit tests, `integration`)?"
  - "Attack-surface question: the unverified degrade (Decision 6) lets a load through without detecting an unknown skill, and the retry step adds a routing edge on the orch pane. Does the card introducing each (cards 1, 2, 3) state in its `**Intent:**` what it lets through and what bounds it?"
  - "Test economy, cards 1 to 3: does any planned new test or row duplicate an existing skill-load, reload or restart test that the card also says it extends (`startup_test.go`, `session_test.go`, `watcher_test.go`), or add one test per symbol rather than at the public surface?"
---

Seeded from the plan's four cards and the decision record's Decisions 1 to 14.
No `Do not flag` item is promoted: the mechanical checks, derived dependency edges and `ImpactSummary` on non-`Edit`/`Delete` cards are left to the gate and the rubric.
