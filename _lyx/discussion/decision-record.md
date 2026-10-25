# Decision record — plan-refs-auto-rewrite

## Goal

Two kinds of stale plan reference are fixed mechanically instead of by hand or by a full Plan perch:
a glyph a Webster fork renamed, named by a card not yet begun,
and a card reference that `lyx loom insert-card` or `remove-card` leaves naming a moved card's old number.

## Scope

- The fork-return report gains a `renames` field: a list of `{old, new}` glyph pairs the batch renamed.
- Record-batch and recover-batch pass the reported renames into the post-batch drift detection, which applies each to the pending cards through the exact tier's repair path: plan-wide `RewriteRefs`, the revalidating resolve, one amendment per repair, and the existing re-baseline.
- The implementer stencil tells the fork to report every glyph it renamed under `renames`.
- `insert-card` and `remove-card` rewrite every card-id reference to a moved card, in the `<N>-<slug>` form and the zero-padded file form `<NN>-<slug>`, in the overview and every card file, and report each rewrite.
  Bare-number prose mentions stay reported and unrewritten.
- Docs: `internal/websterengine/doc.go`, `internal/planglyph/doc.go` where it describes drift, `internal/loomcli/doc.go` and the verbs' `--help`, `contracts/specs/loom-plan-spec.md`'s numbering section.
- Out of scope: the loom recipe and its `on_reroute`, a fixer agent, the digest's `deviations` file list, rewriting bare card numbers.

## Decisions

### Reported renames travel in the report and the digest

Decision: `Report` gets `Renames []Rename` under the YAML key `renames`, each with `old` and `new`.
`ParseReport` refuses an entry with an empty side or with `old` equal to `new`.
`distill` copies the renames into the digest under `renames`, so recover-batch, which reads the strand's report into a digest, feeds the same list to the post-batch pass as record-batch does.
Rationale: the fork is the only party that knows what it renamed, and the report is its one return channel; carrying it on the digest keeps both terminal paths identical, as the post-batch pass already is.

### Reported renames join drift detection's repair set

Decision: `planindex.Delta.DetectDrift` and `planglyph.DetectDrift` take the reported renames.
Each reported pair passes the same two gates as an exact-tier rename: a pair a Rename card declares is that card's own outcome and is skipped, and an old glyph no pending card references is logged only.
A reported pair whose old glyph quarry's exact tier already renamed is skipped, so the AST-proven answer wins.
A surviving pair joins the same substitution map, plan-wide rewrite, revalidating resolve of the introduced glyphs and amendment loop as the exact tier, with the amendment tier word `reported`.
Its old glyph is excluded from the deleted-symbol sweep and from the evidence-tier candidate findings, as an exact pair's constituents are.
Rationale: the operator's decision names that path; reusing it gives a reported rename the same revalidation and audit trail with no second rewrite route.
Bound: a reported rename rewrites only references to its old glyph in the plan's cards, and only when the new glyph then resolves does it pass silently;
a new glyph that does not resolve is a blocking finding, recorded as a later-card warning and refused at that card's begin-batch, which reaches Plan-Burler-Review through the unchanged `on_reroute`.
A missing glyph with no reported replacement stays the existing `plan-references-deleted-symbol` finding.

### Card ids are the unambiguous renumber reference

Decision: the renumber rewrites a token `<N>-<slug>` or `<NN>-<slug>` when `N` is a moved card's old number and `<slug>` is that card's slug, bounded on both sides so it is not part of a longer slug, path segment or number.
The rewrite keeps the token's padding: an unpadded id stays unpadded, a zero-padded one is padded to two digits.
Every substitution is computed from the pre-renumber numbering in one pass, so no chain of moves rewrites a reference twice.
Changed files are journaled like the moved card files, so a failed call is still undone whole.
The result gains a list of the rewritten references, and the CLI envelope reports it as `rewritten_refs`.
The prose-mention scan skips a card-id token of the renumbered plan, so a rewritten id is not reported again as a bare-number mention.
Rationale: a number paired with its slug names exactly one card before and after the renumber, which answers the board entry's open question;
a bare number may already mean the new numbering, so it stays a report.

### Added after Discussion (parent, 2026-10-25): A fork keeps the plan's names; a rename off the plan is reported with its reason

Decision: The implementer stencil (webster fork and recovery strand) says the plan's glyph names are the contract: a fork keeps every name a card gives. A rename the plan does not declare is allowed only when the planned name collides with an existing symbol or breaks a language or lint rule, and the fork then reports it in the new rename field with that reason. Webster applies each reported rename mechanically to the cards not yet begun, as part 1 of the entry says.

Rationale: Operator, 2026-10-25: the plan is the contract, so a fork never renames on its own taste; the mechanical rewrite stays for the rare forced rename so it stops nothing.

## Constraints

- `PATTERN-sole-parsers`: the report is parsed only by `ParseReport`, the plan only by `planparser`.
- `PATTERN-tolerant-records`: the digest's new field is additive JSON, and an older state without it reads as no renames.
- `PATTERN-stencil-ownership`: the implementer stencil states the report's fields, read at call time.
- `PATTERN-refusal-way-forward`: the report refusals for a bad `renames` entry name the recover-batch way forward record-batch already appends.
- `PATTERN-test-speed`: tests are `tier1` where they run without quarry or git.
- An older fork's `deviations` key stays accepted and ignored.

## Auto-mode assumptions

- A reported glyph is spelled as quarry's glyph id, the same spelling the exact tier's `From.ID` and `To.ID` carry.
- A fork that renames nothing omits `renames`; the field is optional.
- The card-id rewrite applies only to cards a renumber moves; a reference to a removed card is left as it is.

## Open risks

- A fork that misreports a rename whose new glyph resolves to an unrelated symbol rewrites later cards wrongly; the amendment log records it with tier `reported`, and the later card's own gates catch work against the wrong symbol.
- A prose token that spells a moved card's id by accident is rewritten; the slug makes that unlikely.

## Acceptance criteria

- A report with `renames` parses, and one with an empty side or an identical pair is refused.
- A reported rename whose old glyph a pending card references rewrites that card, appends a `reported` amendment, and yields no `plan-references-deleted-symbol` finding for the old glyph.
- A reported rename whose new glyph does not resolve yields a blocking finding.
- A reported rename matching a declared Rename card pair, one no pending card references, and one the exact tier already covers each make no repair.
- A missing glyph with no reported replacement still yields `plan-references-deleted-symbol`.
- The reported renames reach drift detection from both record-batch and recover-batch.
- `insert-card` and `remove-card` rewrite padded and unpadded card-id references to moved cards in card files and the overview, leave a longer slug or a non-moved card's id untouched, report each rewrite, and no longer report a rewritten id as a prose mention.
- A failed renumber after a reference rewrite restores every file it changed.

## Notes for the plan writer

- `postBatchChecks` in `internal/websterengine/recordbatch.go` is the one place both terminal verbs reach drift detection.
- `applyRenumber` in `internal/planparser/renumber.go` already journals the overview and the moved cards; unmoved card files join the journal only when the rewrite changes them.
