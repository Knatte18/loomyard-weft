# Card 3 — A per-handle bind outcome from BindHandles

**Create:**
- `plan:internal/planindex#BindReason` -> `type BindReason string`
- `plan:internal/planindex#BindNoDeltaMatch` -> `const BindNoDeltaMatch BindReason = "no-delta-match"`
- `plan:internal/planindex#BindSiblingUnmatched` -> `const BindSiblingUnmatched BindReason = "sibling-unmatched"`
- `plan:internal/planindex#BindOutcome` -> `type BindOutcome struct`

**Edit:**
- `internal/planindex#Delta`
- `internal/planglyph#BindHandles` -> `func BindHandles(plan *planparser.Plan, planDir string, delta quarry.GitDeltaAnswer, cards []planparser.Card) ([]Finding, []planindex.BindOutcome, error)`
- `internal/planglyph#batchDelta.BindHandles` -> `func (d batchDelta) BindHandles(plan *planparser.Plan, planDir string, cards []planparser.Card) ([]Finding, []planindex.BindOutcome, error)`
- `internal/websterengine_test#fakeDelta.BindHandles` -> `func (d fakeDelta) BindHandles(plan *planparser.Plan, planDir string, cards []planparser.Card) ([]planindex.Finding, []planindex.BindOutcome, error)`
- `internal/websterengine#postBatchChecks`
- `internal/planglyph#TestBindHandles_MatchedHandleRewritesDeclaringAndReferencingCard`
- `internal/planglyph#TestBindHandles_PartialMatchOnOneCardMismatchesAndRewritesNeither`
- `internal/planglyph#TestBindHandles_ZeroHandleCardNoFindingNoWrite`
- `internal/planglyph#TestBindHandles_SubstitutionReachesCardOutsideTheCompletedBatch`
- `internal/planglyph#TestBindHandles_RenameNewSideHandleBinds`
- `internal/planglyph#TestBindHandles_RenameFileSidePairIsNotAHandle`
- `internal/planglyph#TestBindHandles_RenameNewSideUnmatchedMismatches`
- `internal/planglyph#TestBindHandles_LanguageNoneNoOp`
- `internal/planglyph#bindCardOne`
- `internal/planglyph#TestRealGitDelta`

**Intent:**
Decision D2: `BindHandles` returns what happened to each handle next to its findings, through a type `planindex` declares, and never logs.
Card 5 logs these outcomes.

Outcome types, in `internal/planindex/planindex.go`:
- Declare `BindReason`, a string type for the closed set of reasons a handle failed to bind.
  It has the two reasons `BindHandles` computes: `BindNoDeltaMatch` (`no-delta-match`) and `BindSiblingUnmatched` (`sibling-unmatched`).
  Card 5 adds the other four reasons as `BindReason` constants in `internal/websterengine`.
- Declare `BindOutcome` with these fields:
  - `Card int`: the card's number.
  - `Handle string`: the handle as the card spells it.
  - `Glyph string`: the glyph the handle bound to, empty when it failed.
  - `Reason BindReason`: empty when it bound.
  Its doc comment says that exactly one of `Glyph` and `Reason` is set.
- The types use only the standard library, so `planindex` keeps its import set.
- Change the `BindHandles` method of the `planindex#Delta` interface to return `([]Finding, []BindOutcome, error)`.
  It is an interface method, so it takes no re-sign arrow.

`planglyph.BindHandles` returns one outcome per handle `cardOwnHandles` lists for each card in `cards`, in card order and then handle order.
Card 4 moves that rule into `planparser.OwnedHandles` without changing what it lists.
- A matched handle on a card whose handles all matched gets the expected glyph it binds to.
- A handle that matched nothing gets `no-delta-match`.
- A handle that matched while another handle of its card did not gets `sibling-unmatched`.
- A card with no handles contributes nothing.
- Under `language: none` there are nil outcomes, as today's nil findings.
- On a `RewriteRefs` error, return the findings and outcomes computed so far beside the error.
The findings, the rewrite and the bind-count-mismatch rule stay unchanged.

Forward the outcomes through `batchDelta.BindHandles` and the test fake `fakeDelta.BindHandles` in `internal/websterengine/gitfake_test.go`.
`postBatchChecks` takes the new middle result and ignores it on this card.

Tests in `internal/planglyph/handle_test.go`:
- Extend `TestBindHandles_MatchedHandleRewritesDeclaringAndReferencingCard` to assert the bound outcome and its glyph.
- Extend `TestBindHandles_PartialMatchOnOneCardMismatchesAndRewritesNeither` to assert `sibling-unmatched` on the matched handle and `no-delta-match` on the other.
- Extend `TestBindHandles_ZeroHandleCardNoFindingNoWrite` and `TestBindHandles_LanguageNoneNoOp` to assert no outcomes.
- `TestBindHandles_RenameNewSideUnmatchedMismatches` asserts the `no-delta-match` outcome.
- The other listed tests and the integration-tagged `bindCardOne` and `TestRealGitDelta` only take the extra result.

**ImpactSummary:** the `planindex` `Delta.BindHandles` contract gains a result; its implementations (`planglyph#batchDelta` and the websterengine test fake) and its one production caller (`postBatchChecks`) change on this card, with no behavior change outside `BindHandles`' new result.
