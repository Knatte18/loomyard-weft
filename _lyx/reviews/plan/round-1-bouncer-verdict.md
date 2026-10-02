---
verdict: APPROVED
rationale: "All six round-1 findings are fixed in the plan as written (card 8 LegacyLoomDriverStrandName on both driver lookups, card 10 standalonegeom naming integration test, card 2 ReadCode treating a malformed code as absent, card 1 Resolve way forward, cards 8 and 11 ImpactSummaries), and spot checks of hubgeom.ReedGeometry callers, reed test fixtures and verify coverage found nothing new."
---

# Plan-Review round 1 — judge verdict

The round's report raised six findings (two MEDIUM, four LOW) under an `APPROVED` reviewer verdict, and the fixer report claims all six fixed.
Each fix is present in the current plan files:

- **F1:** card 8 adds `loomengine.LegacyLoomDriverStrandName = "loom-driver"`, and both `resolveDriverStrandAction` and `driverAliveFrom` match it exactly beside `agentname.Matches` on `driver`.
  Its tests cover the live and dead legacy driver.
  The overview's Shared Decision on `Matches` now states the driver exception.
- **F2:** card 10 creates `internal/standalonegeom/naming_integration_test.go`, which builds the engine from `standalonestate.Derive` and `standalonegeom.ReedGeometry` on an uncoded checkout.
  It asserts `<StandaloneCode(hash8)>:worker`, `LYX_STRAND_NAME` set and `LYX_PARENT` unset.
  `./internal/standalonegeom/...` is in the verify integration list.
- **F3, F6:** cards 8 and 11 now conclude on their blast radius instead of restating Intent.
- **F4:** card 2's `ReadCode` and the clone probe report a malformed record as not found, so card 7's empty-code refusal and its `lyx fabric code <code>` way forward reach a damaged record too.
- **F5:** card 1's `Resolve` refusal names its way forward, and its test asserts the text.

The report carries no `## Focus departures` section, so there is nothing to ratify or reject.
Its focus verdicts that raised no finding hold up: scope license (focus 3), the session-name seam (focus 4), granularity across cards 7, 9 and 10 (focus 5), and ImpactSummary quality on cards 6, 12 and 15 (focus 6).

Independent checks against the worktree:

- **Card 6:** every non-comment use of `hubgeom.ReedGeometry` is in its Edit list.
  The other hits are doc comments (`reedengine/server.go`, `lock.go`, `shuttleengine/run.go` and their tests).
- **Card 7:** every reed test that reaches `AddStrand` builds its engine through `newTestEngine`, `newColdScratchEngine`, `newIntegrationEngine` or the contract test's literal, which card 7 gives a code.
  The one exception is reedcli's `watchdogreap_integration_test.go`, which uses `watchdogIntegrationEngine` over `hubgeom.ReedGeometry` on a `hubforge` hub, coded by card 2.
- **`AddSpec.Round`:** every user (`reedcli/add.go`, `shuttleengine/run.go`, `run_test.go`, `strand_test.go`) is in card 7's Edit list.
- **Verify coverage:** the `## verify:` section's `go test ./...` plus the integration list covers every package a card targets.
  Smoke-tagged edits are compiled by `go vet -tags smoke ./...`.
