---
round: 1
focus:
  - "Fidelity: check every Decision and Constraint in `_lyx/discussion/decision-record.md` (D1 to D4 and any others) is carried by card 1 or 2, and that no card scope, such as the `internal/orchengine/doc.go` edit or the `Short`/`Long` rewording, lacks a licence there."
  - "Card 2 granularity: it creates many private symbols (`orchStateCache.fail`, `overviewTokens`, two consts, `orchStatePoll`); settle which are inseparable supporting members of another card's symbol and which are real public surface."
  - "Card 2 test economy: settle whether `TestOverviewSource_DrawsTheOrchLine` and `TestLoadOverviewThresholds` repeat behavior `TestRenderOverview` or an existing overview source test already covers, naming that test."
  - "Card 2 `Edit` targets: for each Edit glyph whose `Intent` changes parameters, results or receiver (`renderOverview`, `newOverviewSource`), settle whether the re-sign arrow is present, and whether the Edit glyphs for the doc-comment-only changes (`overviewSource.frame`, `battenCLI.overviewCmd`) and the file paths `cli.go` and `doc.go` fit the member-glyph rule."
  - "Card 2 reachability: settle whether each refusal or guard branch it prescribes (failed stat, second strike, `runs unreadable` prepend, `PeekState` not-found race) has a named input and test that reaches it, and whether any named input is rejected by an upstream check."
  - "Card order and verify: card 2 uses `plan:internal/orchengine#PeekState` from card 1; settle that the order holds, and that `## verify:` runs the tests of every package either card targets, including `cmd/lyx` for the test-budget file."
  - "Test-budget edits: settle that the row numbers (143 to 144, 81 to 83) match the current `cmd/lyx/testdata/test-budget.yaml` and the new top-level test counts."
---

Seed focus from reading the plan.
The mechanical checks and the `Do not flag` list are left to the gate and the rubric.
