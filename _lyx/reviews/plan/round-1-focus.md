---
round: 1
focus:
  - "Card 1, `TestStart_LoneEOLMarkBoxTakesStartupSends` and its `start_test.go` target: does an existing `claudeengine` test (`corpus_test.go` or another start-driving test) already cover a ready-box start through Claude's own box reading, so that a new top-level test is a test-economy finding?"
  - "Card 4, `TestStartup_GateDismissalGap` and the `TestStartup_ReadyProbeTiming` rows: does each new row exercise behavior an existing table in `startup_test.go` already covers, and is the new top-level test justified against `PATTERN-test-economy`?"
  - "Card 4 `Edit` member glyphs: `Run.awaitStartup`, `Run.checkLivenessTick`, `playInputs`, `startupTickCap` — does the `Intent` change any parameter, result or receiver of a member whose glyph lacks the re-sign arrow (`playInputs` delegating to `playInputsSent`, `startupTickCap` taking a different interval)?"
  - "Cards 1, 2, 3: `Edit` cards that list `Result`, `Runner.RunGated`, `Run.abandonStartup`, `paneTail` and `mapOutcome` — do the glyphs match what each `Intent` changes, and does any card name symbols while targeting only non-test Go file paths?"
  - "Card 4 against the decision record: the parent's backoff decision replaces D5's exemption, so check that no card amends `PATTERN-daemon-wakeups` and that the backoff base, doubling-while-unchanged and reset-on-change rule are all carried; check that the throttle's `Wait` remark and the debug log are licensed scope."
  - "Card 4 attack-surface and refusal reachability: the dismissal throttle removes key presses; confirm the `Intent` states what it can skip and what bounds it (startup deadline, tick cap), and that each named test row actually reaches the held-dismissal branch and the first-key-failed branch."
  - "Card 3 refusal reachability: each of the three `abandonStartup` tail conditions has a named test row that reaches it, and the 'dies during skill-load' row and the all-captures-failing row are not rejected by an upstream check before they reach the branch."
  - "Card 5 and the `## verify:` section: every package a card targets (`claudeengine`, `shuttleengine`, `shedadapters`, `cmd/lyx` testdata) has its tests run, and the section follows the plan template's order."
  - "Card order and `Shared Decisions`: card 3 relies on card 2 for the producer message, and card 4 touches `startup_test.go` after cards 2 and 3; confirm no card depends on something a later card introduces."
---

Seed focus only; each entry names where to look and a question to settle.
None restates a check the round's own gate enforces.
