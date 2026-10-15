---
round: 1
focus:
  - "Cards 6-9: does any shuttle caller still pick an engine other than from the run record (or from the spec at Start and Open)? Look for leftover hardcoded Claude engine construction outside `internal/shuttleengines`."
  - "Card 13: after `ModelSwitchSequence` is deleted, do any callers, mocks, fakes, docs or test fixtures still reference it? Check the Delete group's `assert-no-callers` ran against every target."
  - "Card 11 and card 12: is each optional capability (fork audit, resume-context rendering, instruction files) asserted at wiring time, and is the absent-capability path handled rather than a panic or silent skip?"
  - "Card 10: does `Spec.Requires` fail at both Start and attach with an error naming the engine and the missing requirement?"
  - "Card 14: does the widened `PATTERN-shuttle-provider-seam` scan match what the diff changed, and is `PATTERN.md` updated in the same commit?"
  - "Test economy: new top-level tests against existing coverage per package, and the `cmd/lyx/testdata/test-budget.yaml` rows for any package pushed past its budget."
  - "Docs: are the `doc.go` files and `docs/overview.md` updated for the new registry package and the shrunken `Engine` interface?"
---

Focus covers the registry seam, the capability moves and the deletion sweep, which are the places a plan-conformant diff is most likely to leave stale callers or docs.
