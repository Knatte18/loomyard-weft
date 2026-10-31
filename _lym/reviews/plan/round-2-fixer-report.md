# Fixer report — plan, round 2

All five findings were fixed.
`lym loom validate-plan` answers `ok: true` over the changed plan, with informational findings only.

## Changes

- **F1** (BLOCKING), fixed as proposed: `05-sampler-registry-entry.md` lists `internal/shedrecipe#TestNames` under **Edit:**.
  Its Intent now says the `ShipsExpectedEntries` subtest gains `"SamplerSession"` in its sorted list, between `"RecordWrite"` and `"SeedChild"`.
- **F2** (LOW), fixed as proposed: `06-sampler-recipe.md` lists `internal/loomrecipe#AmendmentTarget` under **Edit:**.
  Its Intent now has the `AmendmentTarget` doc name `shedrun.RecipeSampler` among its recipe names.
- **F3** (LOW), fixed as proposed: `07-sampler-vocabulary.md` lists `internal/battencli#childSeedParams` and `internal/battenshed#InnerRunDeps` under **Edit:**.
  An Intent sentence adds sampler to the `childSeedParams` doc's `params.parent` list and to the `HandBack` field doc of `InnerRunDeps`.
- **F4** (MEDIUM), fixed as proposed and then reworded by the self-check: the D9 sentence of `03-sampler-stencil.md` no longer names `docs/sandbox-howto.md`.
  It now names the sandbox hub on lym's own repository that the operator names in the pane.
  A new sentence says D9 names the hub by that path, which the writer reads as its source.
  It also says the stencil carries no bare `docs/`, `contracts/` or `internal/` file citation, because `TestStencils_NoBareCrossRepoCitations` refuses one.
  The path stays under **Uses:**.
- **F5** (LOW), fixed as proposed: the ImpactSummary lines of cards 2, 5, 6, 7, 8 and 11 now match their Intent.
  - Card 2 says a test row for each accessor.
  - Card 5 names the name-pin change, the production change to `checkLoomContract`'s text, and that widening `LoomBuildEngines` reaches `verifyGatedRows`.
  - Card 6 names the `InterruptPolicies` entry.
  - Card 7 names the agent-facing stencil text.
  - Card 8 names the callers of `codeBudgetZero` and `rerouteReviewed`, `darnDeps`' new argument, `wireDarn`'s dropped `budgets`, and the two new exported functions.
  - Card 11 names the `loomcli/doc.go` sentences.

## Self-check

- fix-consistency, LOW: card 3 no longer carried the `docs/sandbox-howto.md` path that D9 cites. Fixed with my own change: card 3 now says D9 names the hub by that path, which the writer reads as its source. The decision record lies outside the write surface and needs no change.
- fix-claims, MEDIUM: "carries no `docs/`, `contracts/` or `internal/` path" overstated the scan, which refuses only bare `.md` or `.go` file tokens. Fixed as proposed: the text now says "bare … file citation".
- fix-proposal, LOW: removing the path left the agent no way to find the sandbox hub, and left a broken phrase. Fixed with my own change: the stencil names the sandbox hub that the operator names in the pane. That points the agent at the operator it already works with in the pane and adds no ask-the-operator instruction.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
