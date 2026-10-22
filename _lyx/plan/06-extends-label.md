# Card 6 — Extends type label

**Create:**
- `plan:internal/planparser#CardTypeExtends` -> `const CardTypeExtends CardType = "Extends"`
- `plan:internal/planparser#extendsLabel` -> `const extendsLabel = "**Extends:**"`
- `plan:internal/planparser#checkExtendsOverlap` -> `func checkExtendsOverlap(plan *Plan) []ValidationError`
- `plan:internal/planparser#checkExtendsResign` -> `func checkExtendsResign(plan *Plan) []ValidationError`
- `plan:internal/planparser_test#TestValidate_ExtendsChecks` -> `func TestValidate_ExtendsChecks(t *testing.T)`

**Edit:**
- `internal/planparser#typeLabels`
- `internal/planparser#cardLabels`
- `internal/planparser#TargetGroup`
- `internal/planparser#parseCardBody`
- `internal/planparser#parseTypeLabelCase`
- `internal/planparser#parseEditField` -> `func parseEditField(labelLine, label string, lines []string, start int) (refs []string, resigns []CardResign, next int, err error)`
- `internal/planparser#checkCardMissingField`
- `internal/planparser#checkPathMissing`
- `internal/planparser#CheckEditTestFile`
- `internal/planparser#validate`
- `internal/planparser_test#TestParsePlan_CardFields`
- `internal/planparser_test#TestCheckEditTestFile`
- `internal/planparser/doc.go`
- `contracts/specs/loom-plan-spec.md`

**Intent:**
D3: `**Extends:**` joins the type labels; the card only adds to each ref it names and changes nothing already there.

Parsing: `extendsLabel` joins `typeLabels` (mapping to `CardTypeExtends`) and `cardLabels`, and `parseCardBody` gains its case beside `**Edit:**`; no existing label is a prefix of it.
`parseEditField` takes the label it parses, so `parseTypeLabelCase` routes both `**Edit:**` and `**Extends:**` through it.
An Extends group's refs join `card.Targets` like an Edit group's; an arrow sub-bullet's left token is its ref and its `CardResign` goes into the group's own `Resigns` only, never into `card.Resigns`, so no re-sign check or caller-coverage reader treats it as a re-sign.
`TargetGroup.Resigns`' comment says it is populated for Edit and Extends groups.

Readers that treat an Edit group's targets read an Extends group alike: `checkCardMissingField` requires `ImpactSummary` on a card with an Extends group, `checkPathMissing` checks its refs as it checks Edit refs, and `CheckEditTestFile` reads Extends groups too, so a `_test.go` file self glyph under Extends is refused with the same way forward (the auto-mode assumption).
The flat `Targets` readers, the glyph resolve pass included, see Extends refs unchanged, so an Extends ref that does not exist is `path-missing` or `glyph-not-found` as an Edit ref is.

Two new blocking checks, run from `validate` right after `checkCardFieldOverlap`:
`extends-overlap`, from `checkExtendsOverlap`: one finding per ref that a card names under an Extends group and also under any other type label of the same card, naming the card, the ref and the other label; its way forward is to keep the ref under one label, `**Edit:**` when the card changes what is there.
An Extends ref also under the card's `Uses:` is already `card-field-overlap`, since `Targets` holds Extends refs.
`extends-resign`, from `checkExtendsResign`: one finding per arrow sub-bullet in an Extends group, naming the card and the member, whose way forward is to move that ref under `**Edit:**`, since a re-sign changes what is already there.
Reaching inputs: a card listing `internal/x#F` under both `**Extends:**` and `**Edit:**`, and a card with ``- `internal/x#F` -> `func F(n int)` `` under `**Extends:**`; both pass every earlier check.

`doc.go` in `internal/planparser` and `loom-plan-spec.md` describe the label: the type-label list, a `Extends` row in the Card types table (existing refs, the card only adds, `ImpactSummary` required), the two new checks in the validation-checks table, and the readers that treat it as Edit; the dependency rule follows on card 8.
The spec's sentences that count or list the labels say eight labels, the worked example's "all seven type labels are exercised" included, which becomes "seven of the eight".

Tests:
`TestParsePlan_CardFields` gains rows: an Extends group parsing with `CardTypeExtends`, its refs in `Targets`, and an arrow bullet in the group's `Resigns` but not in `card.Resigns`.
`TestCheckEditTestFile` gains a row: a `_test.go` file self glyph under Extends is refused.
`TestValidate_ExtendsChecks`, table-driven, asserts per row the count of `extends-overlap`, `extends-resign`, `card-field-overlap`, `card-missing-field` and `path-missing`: a clean Extends card; the ref under Extends and Edit; the ref under Extends and Uses; an arrow under Extends; an Extends card without `ImpactSummary`; and an Extends path missing on disk.

**ImpactSummary:** confined to `internal/planparser`: `parseEditField`'s only caller is `parseTypeLabelCase`, re-signed here; the three Edit readers widen to Extends, and no Extends label exists in any plan yet, so no current plan's findings change.
