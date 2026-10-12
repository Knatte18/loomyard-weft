# Merge review after Publish under a parent merge lock

`merge-review` · Task · enhancement, loom, burler, batten

## Seen (2026-10-11)

At PR-Gate the orch forks a reviewer over the whole diff, but Webster-Burler has already reviewed the task's code.
What no review has seen is the interaction with what landed on the parent while the task ran: Publish merges the parent in, and only verify runs over the result.

## Idea (operator, 2026-10-11)

A cold-start agent reviews the merge result after the parent is merged in, inside loom, and only when the merge brought in changes.
The PR-Gate fork then narrows to what needs the orch's context: the run against its board goal and the operator's decisions.

## Design direction (operator and orch, 2026-10-12)

Everything here works for any parent branch, never for a fixed one: `main` is only the parent of most hub runs today.

- The merge review is an optional perch after Publish (a Bouncer review row and a Burler fix row, like every review segment), not a gate inside Publish and not a new producer type: a merge finding nearly always needs a code fix, which a gate has no round for.
  It runs only when Publish's merge brought in changes from the parent.
- A parent merge lock, as Millhouse's `mill-merge` has (`plugins/mill/skills/mill-merge/SKILL.md` in the Millhouse repo: a lock file under the parent's checkout, a stale check, a bounded wait, released on failure and gone stale after a kill): from the moment Publish starts merging the parent in, through conflict resolution, verify, the merge review and PR-Gate, to Land, no other run lands on that parent.
  So the parent cannot move under the review, and no merge or conflict resolution is redone.
  Landing today takes no such lock: only the push or PR merge itself is atomic.
  The lock is per parent branch, needs a bound so a hung run cannot block every other, and another run reaching Publish on the same parent waits for it.
- With the lock held, the PR-Gate fork narrows to what needs the orch's context, so the time the lock is held stays short.

## Merged: `prgate-stale-base` (2026-10-11, bundled 2026-10-12)

`burler-split-budget` landed while `orch-wake-policy` waited at PR-Gate with PR #564.
GitHub then reported #564 `CONFLICTING` against the moved parent: 16 files changed by both tasks.
The PR-Gate review fork would not approve a conflicting PR, so the orch sent the run back with `lyx loom goto --to Publish` and `lyx loom resume`, where Publish merges the parent and its conflict session resolves the conflicts.
The operator's idea: when the parent has moved past the PR's base while a run awaits at PR-Gate, loom sends the run back to Publish by itself.
With the parent merge lock this case should not arise for runs under lyx; the task decides whether a fallback for a parent moved outside lyx (a hand push) is still needed, and whether it checks at PR-Gate entry or also during the wait.

## Open

- Whether the review covers only changes from the parent that touch the task's packages or glyphs, or the whole merge.
- Where in the recipe it runs (after Publish's merge, before PR-Gate) and what a finding sends back to.
- Where the lock lives so every worktree on the hub sees it, and how batten's Land row and a hand landing by the orch take and release it.
