---
verdict: APPROVED
---

## Scope against the board entry

The entry's goal is one deviation rule in the implementer stencil, matching what `ScopeGuard` checks, so the deviation list stops naming files that house the batch's own targets.
The operator's 2026-10-15 decision asks to check the implementer prompt as it stands after `kickstart-per-card` and give it that rule.

- D2 gives `webster-body-implementer.md` one rule, stated once in per-card step 2, and rewords the dangling "every other deviation rule stands" clause to point at it; this is the entry's goal.
- D1 changes `planglyph.ScopeGuard` so file and package targets cover their members.
  Since `30385e1ae` the deviation list comes from the guard, not the fork, so the entry's bug, own target files in the list, is now produced by the guard; stating the guard's current rule would tell the fork its own file-target edits are deviations.
  D1 fixes the bug the entry's title names, at the place it now lives, and is inside the goal.
- D3 replaces the plan spec's stale description of the same comparison with a pointer to `ScopeGuard`.
  It is a small edit to the other text defining the rule the entry asks for, and no other entry claims it.
- The support log's Turn 1 records the post-`kickstart-per-card` check of the fragments the entry asks for.

Scope Out leaves F12 to `stencil-intent-audit` and the implementer's deviation recording in a Weaver run to the `weaver` note, so the design neither duplicates nor contradicts either entry.
`glyph-use-audit`, where the bug was recorded, is removed from the board.

## Overlap with peers and parent commits

`lyx loom peers` lists no parent commits and two peers, both at row 12 against this run's `own_row` 3.

- `engine-usage-readers`: changes `internal/shuttleengine`, `internal/shuttleengines`, `internal/testkit/shuttlefake` and `pattern/PATTERN-shuttle-provider-seam.md`; none of this run's files or behaviour. No overlap.
- `card-own-entry-edit`: its decision record and plan change `CLAUDE.md`, `docs/overview.md`, the loom Discussion and Plan stencils, the darn write stencil and three review rubrics.
  The decision record's open risk expects it to edit `contracts/specs/loom-plan-spec.md` too, but neither its record nor its plan names that file, and it lists no changed paths yet. No overlap.
  Should it touch the plan spec after all, it would be text-only at a different paragraph; `card-own-entry-edit` at row 12 would land first and this run would take its changes at Merge-In.
