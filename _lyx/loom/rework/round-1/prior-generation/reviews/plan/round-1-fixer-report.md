# Fixer report — plan, round 1

## Changes

- F1, fixed as proposed: card 2's `wordingClaims` bullet in `_lyx/plan/02-writer-fork.md` gives the ended-after-last-file path three claims: "the run counts as finished" for the path, "while the session is live the gate re-prompts it" for its live sub-case and "evaluated once over the files as they stand" for its no-longer-live sub-case.
  The keep-verbatim line now names the live sub-case's phrase.
- F2, fixed as proposed: the `twill-directive.md` bullet says its banner comment lists the required markers and names `writer_fork` and `relaunch_rules` as optional, each filled by `Directive` with one part stencil.
  The `twill-relaunch.md` and `twill-writer-fork.md` bullets each say the stencil opens with a host-neutral banner comment naming its required markers, in the `twill-directive.md` banner style.
- F3, fixed as proposed: the writer-fork row of `TestDirective_RendersEveryValueVerbatim` no longer wants "spawn no second fork0" or "skip its step 1".
  It keeps the host values, "Spawn one fork, fork0", "LAST-FILE last" and "every file in FILES-DIR".

## Disputed

None.

## Deferred

None.
