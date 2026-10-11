# PR-Gate sends a run back to Publish when the parent moved and the PR conflicts

`prgate-stale-base` · Note · enhancement, loom

## Seen (2026-10-11, `orch-wake-policy`)

`burler-split-budget` landed while `orch-wake-policy` waited at PR-Gate with PR #564.
GitHub then reported #564 `CONFLICTING` against the new `main`: 16 files changed by both tasks.
The PR-Gate review fork found no defect but would not approve a conflicting PR, so the orch sent the run back with `lyx loom goto --to Publish` and `lyx loom resume`, where Publish merges the parent and its conflict session resolves the conflicts.
Publish already resolves parent conflicts; only the parent moving after Publish, while the run waits at PR-Gate, needs a hand step.

## Idea (operator, 2026-10-11)

When the parent branch has moved past the PR's base while a run awaits at PR-Gate, and the PR no longer merges cleanly, loom sends the run back to Publish by itself before anyone reviews it, so the PR the reviewer sees is merged with the current parent and verified.

## Open

- Whether to check at PR-Gate entry only, or also when the parent moves during the wait (a landing's ready snapshot could trigger it).
- Whether a clean but stale base also goes back to Publish, so the verify covers the merged tree.
- Relation to the `merge-review` note: a merge that Publish redoes may need its review.
