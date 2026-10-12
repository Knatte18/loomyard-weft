---
round: 1
focus:
  - "Fidelity, card 16 against the decision record: settle whether the D5 paragraph (`shedrun.ParseSeed ... the report never calls it`) and the acceptance criterion `shedrun.ReadSeed still rejects it` are fully superseded by the `Added after Discussion (parent, 2026-10-12)` entry, and whether the card carries every Decision and Constraint, and its Intent adds no scope beyond that entry."
  - "Reachability, card 16 Tests: settle whether the `badseed` seed `{\"recipe\":\"loom\",\"driver\":\"rust\"}` actually reaches the `ParseSeed` error branch in `readLoomRecords`, or an upstream check (a tag or recipe filter, or a decode step before it) rejects or reroutes it first."
  - "Attack-surface, card 16 Intent: settle whether the stated bound on the changed seed acceptance (driver and params validation now applied to tagged seeds) is accurate and complete, given the guard that goes away is `ParseSeedTolerant`'s looser acceptance."
  - "Member glyphs and test economy, card 16: settle whether the Edit targets name the right members for the symbols the Intent changes (`readLoomRecords`, `TestParseSeed`, `TestCompute_SelectsRuns`), and whether the planned `badseed` row duplicates what an existing test already covers."
  - "Verify coverage: settle whether `## verify:` runs the tests of every package card 16 targets (`internal/shedrun`, `internal/reportengine`) and is in the order the plan template states."
---

Single-card rework generation, so the focus stays on card 16's fidelity to the parent-decision entry and on whether its new test reaches the branch it claims.
