---
verdict: APPROVED
rationale: "All eight round-1 findings are fixed in the current cards (one-predicate tier rule in card 1, data-file-only read assertion and existence-checked legacy reads in card 4, read-path refusal and fold tests, after-save step for RetireLegacy, distinct legacy constants, grounded ImpactSummaries, semantic-line-break carriage in cards 6 and 10), and a fresh pass against the decision record finds no remaining gap."
---

The report carries no `## Focus departures` section, so there is nothing to ratify or reject.

## Round-1 findings, checked against the current plan

- **F1 (tier-rule exemption):** fixed.
  `01-entry-schema.md:35`–`38` states one predicate per edge, exempting only a `done` dependency, and direction 2 now covers `done` dependents.
  `:55` adds the test for demoting a dependency of a `done` dependent.
  Migrated data cannot violate the new rule on load, because the old `validateWrite` (`internal/boardengine/store.go:131`, `:186`) already refused any non-deferred task depending on a deferred one, with no `done` exemption, and the two legacy stores could not reference each other.
- **F2 (byte-identical read vs. swaplocks):** fixed.
  `04-board-json-store.md:30`–`32` reads only the legacy files that exist, and `:56`–`57` compares data files only, excluding `*.swaplock`.
- **F3 (read-path coverage):** fixed at `04-board-json-store.md:59`–`60`.
- **F4 (RetireLegacy ordering):** fixed.
  `04-board-json-store.md:39` adds the after-save step under the lock, and `05-board-verbs.md:20`–`21` uses it, which matches Decision 10's order.
- **F5 (duplicate constant declarers):** fixed at `03-legacy-migration.md:15`–`16` and `04-board-json-store.md:40`.
- **F6 (card 4 ImpactSummary):** fixed at `04-board-json-store.md:71`.
- **F7 (semantic line breaks):** fixed at `06-render-roadmap.md:22` and `10-overview-doc.md:10`.
- **F8 (card 6 ImpactSummary):** fixed at `06-render-roadmap.md:48`.

## Fresh pass

- **Fidelity:** every Decision (1–23), Constraint and acceptance criterion maps to a card or a Shared Decision.
  "Bolt" in `internal/boardengine` names only the weft-main writer (`fabricengine.Bolt`), as the constraint allows.
  No card adds unlicensed scope.
- **Granularity:** unchanged since the report's analysis, and still sound.
- **Verify coverage:** every targeted Go package is run untagged and with `-tags integration`, and `smoke` is compiled through `go vet`.
- **Non-blocking observation:** card 4's ImpactSummary says `internal/ideengine`'s legacy-seeding tests keep passing.
  `menu_test.go` and `spawn_driven_hub_integration_test.go` are `integration`-tagged, and `ideengine` is outside the integration line of `## verify:`, so only the untagged `spawn_test.go` exercises that claim.
  `ideengine` is not a targeted package, so the rubric's verify-coverage rule does not reach it, and this is not a finding.
