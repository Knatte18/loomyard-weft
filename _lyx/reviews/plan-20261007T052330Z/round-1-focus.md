---
round: 1
focus:
  - "Fidelity: walk every Decision and Constraint in `_lyx/discussion/decision-record.md` and name the card that carries it; settle whether any card adds scope the record does not license."
  - "Attack surface: Card 3 (repeated Enter), Card 4 (halt skips reflection), Cards 5 to 7 (strand removal), Card 11 (watcher reload gating), Cards 14 and 15 (never attach), Card 16 (wait for stopping watcher), Card 17 (re-prompt loop) and Card 18 (new verb); settle whether each `**Intent:**` states what the change can skip, remove or let through and what bounds it."
  - "Test economy: for each card's planned tests, settle whether an existing test already covers the behavior or whether a test per symbol replaces a public-surface test; name the covering test."
  - "Granularity: settle whether Cards 1, 2, 3, 10, 11, 12 and 13 split the provider-seam work into independently reviewable units, or bundle a private helper or constructor into the wrong card."
  - "ImpactSummary: for every `Edit` and `Delete` card, settle whether the `ImpactSummary` is a real blast-radius conclusion rather than a restatement of `Intent`."
---

Round 1 seed from the overview alone.
The `verify:` section runs `go test ./...` and both tagged sets, so package coverage is not a seeded question.
