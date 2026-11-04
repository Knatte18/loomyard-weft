## Goal

```markdown
## Seen (operator, 2026-11-03)

On the NORCE Models repository, on machine OSL-1046, a direct commit or push to `main` is refused by branch protection: every change reaches `main` as a merge from a branch, on the remote.
That is intended and stays so.
lym lands every run by merging locally and pushing the parent: a task's Finalize squashes onto the parent and pushes it, and a beam's pawl run lands the beam the same way.
So on such a repository no task and no beam can land.

## Design (operator, 2026-11-03)

A hub, or a parent branch, can be marked as landed through the remote:

- Finalize pushes the task or beam branch, opens a pull request into the parent and squash-merges it on the remote at once, then pulls the parent, in place of the local squash and push.
- A pull request is always opened, by the hub's own rule, though the protection demands no approval and no status check: lym merges it without waiting for a review.
- The merge is always a squash.
- The parent lock, the gate of the merged tree and the board's done mark stay as today.
- A refused push to a protected branch names this setting as the way forward, instead of a bare push failure.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/code/round-2-review.md; fixer report: _lym/reviews/code/round-2-fixer-report.md.
  - merge-parent-reopen-before-owner-checks (class design, severity MEDIUM, open in rounds 2)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
