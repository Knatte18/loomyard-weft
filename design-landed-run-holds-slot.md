# A landed run left failed before teardown holds a max_runs slot until a hand cleanup

`landed-run-holds-slot` · Task · bug, dobby, batten, cleanup · running · Seed-IDE

Depends on: [`dobby-raddle-ignores-deps`](design-dobby-raddle-ignores-deps.md)

## Seen (orch, 2026-11-11)

`reed-slack-relay` landed on 2026-11-05 and its board entry is done, but its batten run stopped as `failed` at Run-Shed before Worktree-Teardown, with the error `origin ended "reed-slack-relay" (it landed and its board entry is done); run "lym cleanup --apply" from the prime`.
Dobby counts halted runs under `max_runs`, so the landed run held a slot for six days and a released task waited on `max_runs 8` behind it.
Nothing cleaned it up on its own; only a hand `lym cleanup --apply` removes the pair.

## Idea

- Dobby gives no slot to a run whose board entry is done or gone, whatever batten's state says.
- Dobby, or the prime's reconcile, tears down the pair of such a run itself, as `lym cleanup --apply` does, instead of leaving a failing pass for the orch.