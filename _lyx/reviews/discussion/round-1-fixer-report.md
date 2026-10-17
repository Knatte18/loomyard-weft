# Fixer report — discussion round 1

## Changes

All changes are in `_lyx/discussion/decision-record.md`.

- F1, fixed as proposed: D3's Bound no longer says the orch's rule is the only route that writes a decision note.
  It now states that an operator-written note without the `Decision for <slug>:` title, such as one the bouncer escalation stencil counts by its body, no longer moves the snapshot, and that an operator who wants the approve held titles the note in the form or uses `lyx loom decision add`.
- F2, fixed as proposed, with wording corrected by the self-check: the Goal gains "for a run whose seed carries its start stamp".
  D1's Bound gains that a new pair for a reused slug never reaches the zero bound, given the first Auto-mode assumption's cleanup, and that only a pre-`started_at` seed, a removed seed or a hand-edited stamp reaches it.
- F3, fixed as proposed: D2's clause and the matching acceptance criterion now read "close that pull request on GitHub, then resume the run with `lyx loom start`".
- F4, fixed as proposed: D3 renders the title form from `internal/alignment`'s declaration through a template key in the orch role stencil, filled with the literal `<slug>` placeholder because the stencil is hub-wide.
  The key joins the role render's required markers.
  The `PATTERN-producer-pointer-rule` Constraints line is deleted, and the stencil criterion now reads "The rendered orch role stencil names the decision-note title form".
  Scope's stencil line now also names `internal/orchengine`'s role render.
- F5, fixed as proposed: the Open risk now also names PR-Gate, PR-Align, Finalize's close and the approve and reject verbs finding no pull request, and PR-Gate's nil reason that cannot help.
- F6, fixed as proposed: the `PATTERN-leaf-packages` Constraints line is deleted.

## Self-check

- fix-consistency, MEDIUM: F2's "only a pair whose seed predates `started_at`" contradicted D1's three zero-bound causes and skipped the approve and reject verbs, which do not run the bootstrap. Fixed as proposed: the sentence now says what a reused-slug pair never reaches and lists every route that does reach it.
- fix-proposal, LOW: F2's "never" depended on the cleanup assumption without saying so. Fixed as proposed: the sentence now names the first Auto-mode assumption.
- fix-proposal, LOW: F4 left open whether the new template key is a required marker. Fixed with my own change: the key joins the required markers, matching how `RenderRoleFile` treats `command_index` and `repository_rules`, so an override without it fails loudly and does not render silently.
- fix-claims: no findings.
- fix-way-forward: no findings.

## Disputed

None.

## Deferred

None.
