# Autonomy takes in new GitHub issues and runs them, so findings do not wait for the operator

`autonomy-intake-runs` · Note · feature, board, orch, dobby

Depends on: [`orch-unattended-rules`](design-orch-unattended-rules.md)

## Seen

Runs file their findings as GitHub issues on the hub repository.
Nobody takes them in unless the operator asks for intake, so under autonomy they lie until morning.
On 2026-11-27 the operator asked the orch to take in and run every incoming issue as part of autonomy, and said a module of its own for autonomy may be needed.

## Operator decision (2026-11-28): autonomy comes in levels of scope

What autonomy may start is a level of its own, beside how far it settles stops (project `self-healing-autonomy`):

1. First level: autonomy acts only on tasks that are already tasks and already released.
   It settles their stops and lands them, and starts no new work.
2. Next level: the orch may also turn notes, and incoming issues, into tasks and start them.

Turning a note or issue into a task is the risky step, since autonomy could start large work the operator would not approve.
Raddle splits are the example: their first draft is regularly reshaped by the operator, as project `self-healing-autonomy` went from eleven parts to five.

## Idea

An autonomy module that owns what the orch now does by hand overnight at the second level:
watch the hub repository for new issues, import them, bundle them by package, chain them against running and waiting tasks,
start them as darn light, and report what it started.
It shares its home with the stop settling that project `self-healing-autonomy` builds.

## Open

- Whether this is lym code (dobby or a new module) or an orch procedure.
- What the second level never starts on its own; likely raddle splits, projects, features, and darn solid or wide.
- How the scope levels combine with `beam.autonomy`'s levels from `self-healing-autonomy`.
