# A run checks what moved on origin and its owners before it pushes or lands

`remote-moved-before-push` · Task · enhancement, fabric, batten · done

## Seen (operator, 2026-11-03)

The operator often works on another machine while this one sleeps, whether the sleep came from `lym hibernate request` or from the machine itself.
When origin or a run's owner moved meanwhile, a run that goes on after the wake works on a stale base, and nothing tells it until a push is refused or a landing conflicts.

## Idea

The check belongs where the stale base does harm, not at the wake (operator, 2026-11-03):

- Before a run commits for a push, pushes, or lands, lym fetches and compares the run's base and parent branch, and the owner of the run, its beam and root, with what the run last saw.
- When an owner moved to another machine, the run stops before it pushes, and the stop names `lym adopt`.
- When only origin's tips moved, the run takes the new base the way a landing already does, or stops naming what moved when it cannot.
- The orch hears what moved in one short notice line.
