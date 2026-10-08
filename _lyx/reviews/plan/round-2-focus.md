---
round: 2
focus:
  - "Test budget: `PATTERN-test-speed` caps each package at the top-level `func Test…` count pinned in `cmd/lyx/testdata/test-budget.yaml`, enforced by `cmd/lyx/testbudget_test.go`. For every card that adds a top-level test function to a package, settle whether the card either raises that package's row in the same card or folds the test in as a row of an existing function."
  - "Test economy in the deny cards: `internal/shuttleengine/claudeengine/settings_test.go` holds `TestBuildDenyNotice_MatchesInstalledDenies` and `prepare_test.go` holds `TestPrepare_AppendSystemPromptMatchesDenyNotice`. Settle whether any card plans a new test or row set for a notice-matches-installed-hooks assertion those two already cover, rather than extending them."
  - "Same-commit docs: for each card that changes observable behavior in `internal/shuttleengine` or `internal/shuttleengine/claudeengine`, settle whether the package `doc.go` and the comments that describe the old behavior are in the card's targets or its Intent."
  - "Coverage of the decision record: settle whether every post-Discussion decision and acceptance row in `_lyx/discussion/decision-record.md` is carried by a card and pinned by a test row where the card plans tests."
---

Focus entries target the plan only; the decision record is the measuring stick.
