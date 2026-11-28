# Autonomy takes in new GitHub issues and runs them, so findings do not wait for the operator

`autonomy-intake-runs` · Note · feature, board, orch, dobby

Depends on: [`orch-unattended-rules`](design-orch-unattended-rules.md)

## Seen

Runs file their findings as GitHub issues on the hub repository, and runs and other agents file notes on the board, such as bugs.
Nobody takes either in unless the operator acts, so under autonomy they lie until morning.
The orch runs intake only when the operator asks (operator, 2026-11-28).

## Operator decision (2026-11-28): autonomy comes in levels of scope

What autonomy may start is a level of its own, beside how far it settles stops (project `self-healing-autonomy`):

1. First level: autonomy acts only on tasks that are already tasks and already released.
   It settles their stops and lands them, and starts no new work.
2. Next level, a fuller autonomy: the orch also picks up the notes that runs and other agents file on the board, and incoming issues,
   decides what to do with each, and may turn them into tasks and start them.

Turning a note or issue into a task is the risky step, since autonomy could start large work the operator would not approve.
Raddle splits are the example: their first draft is regularly reshaped by the operator, as project `self-healing-autonomy` went from eleven parts to five.

## Idea

An autonomy module that owns the second level:
watch the board for new notes and the hub repository for new issues, import the issues, bundle the work by package,
chain it against running and waiting tasks, start it as darn light, and report what it started.
It shares its home with the stop settling that project `self-healing-autonomy` builds.

## Open

- Whether this is lym code (dobby or a new module) or an orch procedure.
- What the second level never starts on its own; likely raddle splits, projects, features, and darn solid or wide.
- How the scope levels combine with `beam.autonomy`'s levels from `self-healing-autonomy`.
