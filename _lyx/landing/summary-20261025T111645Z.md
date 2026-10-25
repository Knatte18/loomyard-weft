# Plan references follow renames and renumbers mechanically

Two kinds of plan reference used to go stale and were then fixed by hand or by a full Plan perch.
This change rewrites both mechanically.

## A fork's reported renames reach the later cards

A Webster fork keeps the plan's glyph names.
The implementer stencil, which forks and recovery strands share, now says a fork renames a planned symbol only when the name collides with an existing symbol or breaks a language or lint rule.
The fork then lists the rename in the batch report's new `renames` field, as an `old` glyph, a `new` glyph and a `reason`:

```yaml
renames:
  - old: internal/gatecli#runGuardStep
    new: internal/gatecli#guardStepRun
    reason: runGuardStep collides with an existing method
```

`ParseReport` refuses an entry that is missing any of the three fields or that names one glyph as both `old` and `new`.
`distill` copies the list onto the digest, so record-batch and recover-batch both pass it to `postBatchChecks`.
From there it goes to `planindex.Delta.DetectDrift`, which now takes the reported renames as an argument.

`planglyph.DetectDrift` runs each reported pair through the same two gates as an exact-tier rename.
A pair that a Rename card declares is skipped, and so is an old glyph that no pending card references.
A pair that passes both gates joins the exact tier's repair path: one plan-wide `RewriteRefs`, the revalidating resolve of the new glyphs and one amendment per repair, with the tier word `reported`.
After that, record-batch re-baselines the plan fingerprint as it already did.
A repaired old glyph gets neither a `plan-references-deleted-symbol` finding nor a `rename-candidate` finding.
This covers the case behind the task: a function renamed and changed in one commit, which quarry leaves in Deleted or as an evidence-tier candidate.
A reported new glyph that does not resolve becomes the post-repair resolve's blocking finding.
That finding is recorded as a later-card warning and refused at that card's begin-batch, so it reaches Plan-Burler-Review through the unchanged `on_reroute`.
A glyph that is missing with no reported replacement is still `plan-references-deleted-symbol`.
The loom recipe is unchanged and no fixer agent is added.

The master stencil's digest field list now includes `renames`, which is informational, and the template test that pins that list follows it.

## Renumbers rewrite card ids

`lyx loom insert-card` and `remove-card` now rewrite every card-id reference to a moved card to its new number.
That covers `<N>-<slug>` and the zero-padded file form `<NN>-<slug>`, in the overview and in every card file.
A card id names exactly one card on both sides of a renumber, because the slug pins it.
A bare number may already mean the new numbering, so a line naming one is still reported under `prose_mentions` and not rewritten.
A rewritten id is masked from that scan, so `--card 9-beta` is not reported again as "card 9".
Each rewrite is reported in `RenumberResult.RewrittenRefs` and in the CLI envelope's `rewritten_refs`, with `file`, `line`, `from` and `to`.
A card that keeps its number but holds a rewritten reference is journaled with the rest of the edit, so a failed call still restores every file.
The plan spec's numbering section now tells writers to refer to another card by its card id.

## Design decisions

- When quarry's exact tier and a fork's report both name the same old glyph, the exact tier's destination wins, because an AST-proven pair is the stronger evidence.
- A reported rename follows the exact tier's order: rewrite, then revalidate.
  The plan is not pre-resolved before the rewrite, so there is only one repair route.
- A card-id token must be bounded on both sides.
  A number inside a longer word, such as `x10-gamma`, or a slug that runs on into a longer one, such as `9-betamax` or `9-beta-two`, is left alone.
- A token keeps its padding: a zero-padded number stays padded to its width, and a token followed by `.md` is the two-digit file form.
  Any other token is unpadded, so `10-gamma` becomes `9-gamma` rather than `09-gamma`.
- A reference to the removed card is left as it is.
- The digest's `deviations` list keeps its meaning as the scope guard's file list.

## Tests

- `TestDetectDrift_RenameAutoRepairsAndAmends` covers both tiers: a reported rename whose old glyph is deleted, one whose old glyph has evidence-tier candidates, and the exact tier winning over a conflicting report.
- `TestDetectDrift_HandBuiltDelta` adds rows for a reported rename that matches a declared pair, one that nothing references, and a deleted glyph that no reported rename replaces.
- `TestDetectDrift_RepairIntoAnUnresolvableGlyphIsReported` covers an unresolvable reported glyph beside the exact-tier case.
- The report tests round-trip `renames` into the digest and refuse malformed entries.
- A tier1 record-batch test and a recover-batch test each show a reported rename that quarry cannot prove rewriting the pending card.
- `TestRenumber_RewritesCardIDs` and `TestRenumber_CardIDRewriteFailureReversed` cover the renumber rewrite, its bounds, padding, the masked prose scan and the reversal.
