# Card 1 — uses-later-target

**Edit:**
- `internal/planparser/validate.go`
- `internal/planparser/validate_test.go`
- `contracts/specs/loom-plan-spec.md`
- `contracts/stencils/loom/loom-rubric-plan-review.md`
- `internal/webstercli/cli_test.go`

**Uses:**
- `internal/websterengine/sequence.go`

**Intent:**
Add decision D1's blocking pure check, `uses-later-target`, to `internal/planparser`'s format set, so a plan whose card order is not executable never passes the Plan-Gate, the Rework-Plan-Gate, `lyx webster validate`, run entry or begin-batch's dispatch validation.

The check refuses a card whose `Uses:` names an entry that appears in the flat `Targets` list (every type label, **Delete:** included) of a card with a higher number.
The match rule is exactly `deriveEdges`' producer-before-consumer rule in `internal/websterengine/sequence.go`: exact string equality after `strings.TrimSpace`, entries that trim to empty ignored on both sides, ref kind not consulted.
It compares cards by number only, whatever batch they would sit in.
Implement it as a new `checkUsesLaterTarget` in `validate.go`, called from `validate` right after `checkCardFieldOverlap`, so both `Validate` and `ValidateFormat` run it.
The finding is attributed to the consumer card (the one whose `Uses:` matches), one finding per (consumer, producer, entry), in card-number then entry order.
Its detail names both cards and the shared entry, and tells the writer to move the work or reorder the cards so the producer comes first, e.g. `card 2 Uses "x", which card 4 targets; a plan runs in card-number order, so move that work or reorder the cards so card 4's change comes before card 2`.
Attribution to the consumer is what makes `planglyph.ValidateDispatch` drop a pair whose consumer card is already begun, with no change there.

No separate cycle check is added: `deriveEdges`' other edge kind (two cards sharing a target) always points from the lower number to the higher, so once no `Uses` edge points backward a cycle cannot form.

Update `validate.go`'s file comment and the `Validate`/`ValidateFormat` doc comments so they name the new check ID and stop pinning a check tally ("twenty-seven", "twenty-eight"): say "every check ID listed above" and "every one but plan-unapproved".
Add the check as a numbered row to `contracts/specs/loom-plan-spec.md`'s "Validation checks" list, right after `card-field-overlap`, renumbering the rows below it and any in-list cross-reference by row number (e.g. "row 25 below").
In `contracts/stencils/loom/loom-rubric-plan-review.md` replace the pinned "twenty-eight"/"twenty-seven of the twenty-eight" tally with wording that does not count the rows.
Leave `loom-plan-spec.md`'s "DAG of intent" prose and `loom-template-plan.md` alone; they stay with the board note `webster-delete-ordering`.

Tests:
in `validate_test.go`, a table row set for `uses-later-target`: a card 2 whose `Uses:` names a target card 4 edits is refused with one finding attributed to card 2 naming cards 2 and 4 and the entry; the same plan with the two cards' content swapped passes; a **Delete:** target of a later card matches; whitespace-padded and empty entries follow `refsIntersect`'s rule; a `Uses:` naming an earlier card's target passes.
Grep the existing planparser fixtures (`internal/planparser/testdata`, inline plans in the package's tests) and the other packages' plan fixtures that run `ValidateFormat`/`Validate` for a `Uses:` naming a later card's target, and reorder any such fixture so its tests keep their original intent.
In `internal/webstercli/cli_test.go`, drive `lyx webster validate` through `c.validateCmd()` over the `skills-in-one-turn` shape (card 2 `Uses:` a file card 4 edits): exit 1, the envelope's findings carry `uses-later-target` under card 2's key naming card 4; the swapped plan exits 0.

**ImpactSummary:** One new pure check in planparser's shared format set, so every plan gate and webster's validate/run/begin-batch refuse a backward Uses edge; spec row and rubric wording follow.
