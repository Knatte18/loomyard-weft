# A bench run creates and removes its own bench pairs, forked from its task worktree

`bench-makes-own-pairs` · Task · enhancement, fabric, darn, batten · starting · Worktree-Create

## Seen (operator, 2026-11-27)

A darn docs bench that needs pairs of its own cannot make them:
the fabric guard refuses every mutating `lym fabric` verb from a task session on the real hub (`internal/fabriccli/roleguard.go`).
Today the bench agent asks the orch, which runs `env -C <task worktree> lym fabric add <pair>` and removes the pairs when told.
The operator finds that hand-off a hindrance: the bench waits on the orch at every step.

## Idea

The guard lets a darn run with profile `docs` create and remove bench pairs itself, within bounds:

- Only `lym fabric add` and the removal verb, run from the task's own worktree, so each pair forks from the task branch, as the orch does it today.
- Only pairs whose slug starts with the task's slug, such as `<slug>-b1`, so a bench never touches another task's pair.
- Worktree-Teardown removes every bench pair the run left behind, so none outlive the run.
- The bench body's cap on runs bounds how many pairs it makes; whether lym also caps the number is open.
