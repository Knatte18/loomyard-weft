# Autonomy takes in new GitHub issues and runs them, so findings do not wait for the operator

`autonomy-intake-runs` · Note · feature, board, orch, dobby

## Seen

Runs file their findings as GitHub issues on the hub repository.
Nobody takes them in unless the operator asks for intake, so under autonomy they lie until morning.
On 2026-11-27 the operator asked the orch to take in and run every incoming issue as part of autonomy, and said a module of its own for autonomy may be needed.

## Idea

An autonomy module that owns what the orch now does by hand overnight:
watch the hub repository for new issues, import them, bundle them by package, chain them against running and waiting tasks,
start them as darn light, and report what it started.
It would share its home with the stop settling in Sketch `self-healing-runs`.

## Open

- Whether this is lym code (dobby or a new module) or an orch procedure.
- Which issues it never runs on its own: features, wide changes, data risk.
- The operator is still thinking this through.
