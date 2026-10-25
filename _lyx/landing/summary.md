# Plan references follow glyph renames and card renumbers mechanically

Two kinds of stale plan reference were fixed by hand or by a full Plan perch although the change is mechanical.
Both are now rewritten in place.

## Fork-reported glyph renames reach the pending cards

A Webster fork that renames a glyph now says so.
Previously `begin-batch` on a later card naming the old glyph refused with `plan_conflict: tree_moved`, and `on_reroute` sent the run through Plan-Burler-Review for a one-glyph fix.

- The fork-return report (`internal/websterengine/report.go`) gains an optional `renames` list of `{old, new, reason}` entries.
  `ParseReport` refuses an entry with an empty `old` or `new`, or with `old` equal to `new`.
  An older fork's `deviations` key is still accepted and ignored.
- The digest carries the renames, so `record-batch` and `recover-batch` feed the same list into the post-batch drift pass (`postBatchChecks`).
  An older digest without the field reads as no renames.
- `planglyph.DetectDrift` and `planindex.Delta.DetectDrift` take the reported renames and run each through the exact tier's repair path: plan-wide `RewriteRefs`, a revalidating resolve of the new glyph, one plan amendment per repair, and the existing re-baseline.
  The amendment tier word is `reported`.
- A reported pair is skipped when a Rename card declares it, when no pending card references its old glyph (logged only), or when quarry's exact tier already renamed that old glyph, so the AST-proven answer wins.
- A reported new glyph that does not resolve stays a blocking finding, refused at that card's `begin-batch` and routed to Plan-Burler-Review as today.
  A missing glyph with no reported replacement is still `plan-references-deleted-symbol`.
- The implementer stencil now tells a fork that the plan's glyph names are the contract.
  A fork renames only when the planned name collides with an existing symbol or breaks a language or lint rule, and reports the rename with that reason.

The loom recipe and its `on_reroute` are unchanged, and there is no fixer agent.

## `insert-card` and `remove-card` rewrite card ids

Both verbs used to report every line naming a moved card and leave the fix to the writer.
They now rewrite a card id that names a moved card, in the overview and every card file, and report each rewrite as `rewritten_refs` (`file`, `line`, `from`, `to`) in the envelope.

- A token `<N>-<slug>` or the zero-padded file form `<NN>-<slug>` is rewritten when `N` is a moved card's old number and `<slug>` is that card's slug.
  The token is bounded on both sides, so a longer slug, path segment or number is left alone.
  The rewrite keeps the token's padding.
- All substitutions are computed from the pre-renumber numbering in one pass, so a chain of moves never rewrites a reference twice.
- Changed files join the existing journal, so a failed call still restores every file.
- Bare-number prose mentions stay reported under `prose_mentions` and are not rewritten, because a bare number may already mean the new numbering.
  A rewritten id is no longer reported again as a mention.
- A reference to a removed card is left as it is.

The pairing of number and slug answers the board entry's open question: it names one card both before and after the renumber.

## Docs

`internal/websterengine/doc.go`, `internal/planglyph/doc.go`, `internal/loomcli/doc.go`, the verbs' `--help`, the webster stencils and `contracts/specs/loom-plan-spec.md`'s numbering section describe the new behavior.

## Risks and checks

- A fork that misreports a rename whose new glyph resolves to an unrelated symbol rewrites later cards wrongly.
  The `reported` amendment records it, and the later card's own gates are the backstop.
- A prose token that spells a moved card's id by accident is rewritten; the slug makes that unlikely.
- Not run by hand: a live Webster run with a fork that reports a `renames` entry, and `insert-card` or `remove-card` on a real plan with ids in prose.
