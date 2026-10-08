# Card 8 — redundant-file-target

**Create:**
- `internal/planglyph/plangate.go`
- `plan:internal/planglyph#planGatePass` -> `func planGatePass(plan *planparser.Plan, worktreeRoot string) ([]Finding, error)`

**Edit:**
- `internal/planglyph#ValidateFormat`
- `internal/planglyph#Validate`
- `internal/planglyph#ValidateDispatch`
- `internal/planglyph/doc.go`
- `internal/planglyph/glyphchain_test.go`
- `internal/planglyph/glyphchain_integration_test.go`
- `internal/planparser/testdata/goodplan/02-json-flag.md`
- `internal/planparser/parse_test.go`
- `internal/planparser/validate_test.go`

**Prosa:**
- `contracts/specs/loom-plan-spec.md`

**Uses:**
- `internal/planglyph#answerSymbols`

**Intent:**
Add D3's resolve-backed half, the blocking check `redundant-file-target` (a card listing a file and a member glyph that resolves into that file), inside a new pass that runs at the plan gates only.

`plangate.go` declares `planGatePass`, the resolve-backed pass `ValidateFormat` and `Validate` call after `resolvePass` (and through `ValidateFormat`, `ValidateRework` and every verb that calls them), and `ValidateDispatch` never calls.
It is not added inside `resolvePass`, which `ValidateDispatch` shares.
Under a non-glyph `language:` it returns nothing and opens no repository.
Otherwise it opens the repository and resolves every card's own member glyph targets (`Targets` only; `plan:` handles excluded by `planparser.IsHandleRef`, self glyphs by `glyph.Parse`) in one batched `resolveTargets` call, which later cards' checks share.
For each card, a file self glyph whose `UnitPath` equals the `File` of a symbol its own member target resolves to is one finding, attributed to the card with `Ref` the member.
Answers are read through `answerSymbols`, so a not-found, ambiguous or unreadable member is skipped here; the status policy already reports it.
The detail names both and the way forward: keep the member glyphs and drop the file, or keep the file when the card changes the whole file.
Infrastructure failures are `ErrQuarryUnavailable`-wrapped, and `ValidateFormat` and `Validate` skip the pass when `resolvePass` already failed.

`ValidateDispatch`'s doc now says a caller with no completed card gets `ValidateFormat`'s answer less this pass.
`doc.go` gains the check's bullet and the rule the pass exists for: a check whose verdict depends on state the run itself changes runs at the plan gates only.

What this lets through: dispatch never refuses on the file-versus-member pairing, so a pairing that only appears once record-batch binds a handle into a member glyph passes.
What bounds it: a redundant listing is plan noise that changes no card's work, and the plan gate already refused every pairing it could see.

The spec gains the row (emitted by `internal/planglyph` at `ValidateFormat`, `Validate` and `ValidateRework`, never at `ValidateDispatch`), renumbered with every `row N` cross-reference fixed.
Its examples stop showing a file beside its own member: the Card fields example's `**Edit:**` lists `internal/boardcli#newListCmd` and `list_test.go`, and the worked example's card 2 drops `list.go` from its `**Edit:**`.
The golden fixture `02-json-flag.md` drops the same bullet, `TestParsePlan_GoldenFixture`'s card 2 expectations follow, and `TestValidate_GoldenFixture_ZeroFindings`' materialization comment names `list.go` as card 3's Uses alone.

`glyphchain_test.go` gains rows: a card listing `shapes/shapes.go` and `shapes#Func` is flagged by `ValidateFormat` and not by `ValidateDispatch`; `shapes/shapes.go` beside `dirname#DirDiffers` is clean.
`glyphchain_integration_test.go`'s `rework-generations` row additionally asserts no `redundant-file-target` at the plan gate or after binding: card 2 lists a file and a handle built in that file.

**ImpactSummary:** Adds one blocking check reachable only from the plan-gate entry points; the golden fixture and spec examples lose one redundant bullet each, and dispatch validation is unchanged.
