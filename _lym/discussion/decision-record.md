## Goal

```markdown
Imported from [issue #278](https://github.com/Knatte18/loomyard/issues/278).

A run was interrupted by a machine reboot at the Darn-Write row and re-stepped.
The re-spawned agent's prompt said the prior work was `(none)` and that the branch held no work of this session's.
The branch already carried three commits from the interrupted spawn, and `.scratch` held a half-finished measurement.
The agent had to read the halt note and file timestamps to tell the earlier spawn's work apart from someone else's.

This is abnormal because the prompt's prior-work statement contradicts the branch state, and an agent trusting it could redo or overwrite finished work.

What to do: when a row is re-stepped after an interrupted-stop halt, compute the prior-work section from the commits the branch holds beyond the base and from the surviving scratch files, or state that the earlier spawn was interrupted and its work may be present.

Provenance: task count-branching-solver; note Darn-Write.md.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - darndeps-prior-work-wiring-untested (class design, severity MEDIUM, open in rounds 1)
  - scratch-files-alone-claim-interrupted-spawn (class design, severity MEDIUM, open in rounds 1)
  - writer-stencil-asserts-commit-ownership (class consistency, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
