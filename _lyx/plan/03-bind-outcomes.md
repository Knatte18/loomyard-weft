# Card 3 — Handle ownership in planparser, and a per-handle bind outcome

**Create:**
- `plan:internal/planparser#OwnedHandles` -> `func OwnedHandles(c Card) []string`
- `plan:internal/planindex#BindReason` -> `type BindReason string`
- `plan:internal/planindex#BindNoDeltaMatch` -> `const BindNoDeltaMatch BindReason = "no-delta-match"`
- `plan:internal/planindex#BindSiblingUnmatched` -> `const BindSiblingUnmatched BindReason = "sibling-unmatched"`
- `plan:internal/planindex#BindOutcome` -> `type BindOutcome struct`

**Delete:**
- `internal/planglyph#cardOwnHandles`

**Edit:**
- `internal/planindex#Delta`
- `internal/planglyph#BindHandles` -> `func BindHandles(plan *planparser.Plan, planDir string, delta quarry.GitDeltaAnswer, cards []planparser.Card) ([]Finding, []planindex.BindOutcome, error)`
- `internal/planglyph#batchDelta.BindHandles` -> `func (d batchDelta) BindHandles(plan *planparser.Plan, planDir string, cards []planparser.Card) ([]Finding, []planindex.BindOutcome, error)`
- `internal/planglyph/doc.go`
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
Decision D2.
The rule for which `plan:` handles a card owns moves into one exported `internal/planparser` function.
`BindHandles` and, from card 4 on, `internal/websterengine` both call it, so the passes that never reach `BindHandles` list the same handles it binds.
`BindHandles` returns what happened to each handle next to its findings, through a type `planindex` declares, and never logs.
Card 4 logs these outcomes.

Ownership (`PATTERN-ref-shape-registry` puts this decision in `planparser`):
- Add `OwnedHandles` to `internal/planparser/handle.go`.
  It returns a card's Create declarations' handles followed by every Rename pair's `New` side that `IsHandleRef` reports as a handle, in body order.
  That is exactly what `cardOwnHandles` returns today.
  Carry over its doc comment, including why a Rename-only card's handle-shaped `New` side counts.
- Delete `cardOwnHandles` from `internal/planglyph/handle.go`, and make `BindHandles` call `planparser.OwnedHandles` in its place.
- Rewrite every mention of `cardOwnHandles` to name `planparser.OwnedHandles`: in `BindHandles`' doc comment and body comments, in `TestBindHandles_RenameFileSidePairIsNotAHandle`'s doc comment, and in `internal/planglyph/doc.go`'s `bind-count-mismatch` entry.
- The existing `BindHandles` tests already pin the ownership rule, so the move needs no test of its own.
  They cover a Rename new-side handle binding, a file-rename pair that is not a handle, and a zero-handle card being skipped.

Outcome types, in `internal/planindex/planindex.go`:
- Declare `BindReason`, a string type for the closed set of reasons a handle failed to bind.
  It has the two reasons `BindHandles` computes: `BindNoDeltaMatch` (`no-delta-match`) and `BindSiblingUnmatched` (`sibling-unmatched`).
  Card 4 adds the other four reasons as `BindReason` constants in `internal/websterengine`.
- Declare `BindOutcome` with these fields:
  - `Card int`: the card's number.
  - `Handle string`: the handle as the card spells it.
  - `Glyph string`: the glyph the handle bound to, empty when it failed.
  - `Reason BindReason`: empty when it bound.
  Its doc comment says that exactly one of `Glyph` and `Reason` is set.
- The types use only the standard library, so `planindex` keeps its import set.
- Change the `BindHandles` method of the `planindex#Delta` interface to return `([]Finding, []BindOutcome, error)`.
  It is an interface method, so it takes no re-sign arrow.

`planglyph.BindHandles` returns one outcome per handle `planparser.OwnedHandles` lists for each card in `cards`, in card order and then handle order.
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

**ImpactSummary:** `cardOwnHandles` becomes `planparser.OwnedHandles` with `BindHandles` its only caller, and the `planindex` `Delta.BindHandles` contract gains a result; its implementations (`planglyph#batchDelta` and the websterengine test fake) and its one production caller (`postBatchChecks`) change on this card, with no behavior change outside `BindHandles`' new result.
