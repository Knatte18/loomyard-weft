---
round: 1
focus:
  - "Fidelity: does every Decision D1-D10 and every Constraint in the decision record map to a card, including the Constraints with no obvious owning card (`PATTERN-mutation-record` partial envelope, the `loom-plan-spec.md` and `doc.go` check-list rows for D1 and D2, refusal-spec rows for D4-D8)? Does any card add scope the record does not license, such as the destructive-guard tally rewording in card 11?"
  - "Card 11 (D7): the largest card, with eleven Edit targets across fabricengine, webstercli, websterengine docs, the destructive guard and refusal-spec. Is it one independently reviewable unit, or does the bundling of the executor, the CLI partial envelope and the guard-table rewording hide a split that granularity calls for? Does its Intent state the bound for the new force-push executor and for the `SkipGit`-ignored remote update?"
  - "Attack-surface: for D3 (Delete target missing becomes an advisory), D6 (rebaseline restamp on failed, dead or stuck batches) and D7 (lyx-performed force update), does each introducing card's `**Intent:**` state what it can now let through and what bounds it, as the decision record does?"
  - "Test economy: for each card's planned tests, does an existing test already cover the behavior, so a new test function or file is redundant? Name the covering test. Check in particular the new `landingdeps_integration_test.go` in card 12 against existing Publish or loomcli coverage, and the new test in card 11 against the extended `ResetPairWarp` tests."
  - "Verify coverage: every package a card targets (including `contracts/`, `cmd/lyx` and any package a card's test targets sit in) has its tests run by a `## verify:` line, tagged integration tests included. Does `go test -tags integration` cover every package that holds a planned `_integration_test.go`?"
  - "ImpactSummary on `Edit` cards: is each a real blast-radius conclusion rather than a restatement of Intent? Check card 10, which uses `Create` plus `Edit` and carries one."
---

Seed focus only; entries name where to look and which question to settle, not verdicts.
