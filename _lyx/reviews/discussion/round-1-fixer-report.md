# Fixer report — discussion, round 1

## Changes

- F1, fixed as proposed: in `decision-record.md`'s Acceptance criteria, the single text-equality sub-bullet is replaced by two.
  One pins each new inline fragment's body against the matching `1e39afc10^` fragment body, banners stripped;
  the other pins the rendered FRESH-READ, locate step and `## Kick-start pack` section against those bodies with `specs_dir` and `pack_cards` filled.
  Premise confirmed: `git show 1e39afc10^:contracts/stencils/webster/webster-fragment-locate-pack.md` ends with ``Card grammar is in `{{.specs_dir}}/loom/loom-plan-spec.md`.``
- F2, fixed as proposed: D4 now states, before its Rationale, that a `<batch>-pack.md` from an earlier file-layout begin-batch of the same batch stays on disk and that the inline prompt never names it.
  Scope Out's last bullet, which said the same, is deleted.

## Disputed

None.

## Deferred

None.
