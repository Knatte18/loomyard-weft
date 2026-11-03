# Drift scope settable in a task's run_settings; agent only when unset

A task can now fix its regression scope in its board entry, so the operator decides it while discussing the task:

```
"run_settings": {"drift": {"scope": "full"}}
"run_settings": {"drift": {"scope": "partial", "skip": ["<test>", "..."]}}
"run_settings": {"drift": {"scope": "off"}}
```

When `drift.scope` is set, Drift-Baseline records it as given and spawns no baseline agent.
When it is empty, the agent decides as before.
Until now the only way to correct a wrong agent choice was to pause after Drift-Baseline, edit `_lym/drift/scope.json`, run `lym fabric commit` and resume, since `lym drift scope` refuses a second call.

Closes #30

## Design decisions

- `scope` and `skip` are new keys of the hub-wide `drift` config module (`internal/driftengine/template.yaml`), empty by default, and are classified as local keys, so a task's `run_settings` layer can carry them.
- `LoadConfig` validates them: `scope` must be empty, `off`, `partial` or `full`; `partial` requires a non-empty `skip`; `skip` under any other scope is refused.
  Each refusal names the key and points at the board entry's `run_settings` and `lym config drift`.
- A scope already recorded, by an earlier pass or by the run's parent at a blocked row, wins over the configured one, so a parent's correction survives a resume.
- With a configured scope and no `baseline.md`, Drift-Baseline writes a generated one that names only the scope and its skipped tests.
  The report says no baseline agent ran, and an existing `baseline.md` is left untouched.
- The Drift-Verify stencil (`drift-template-verify.md`) now tells the verify agent that such a report holds no per-test expectation.
- The capture step after the scope is recorded is unchanged: scope `off` takes none, `full` and `partial` take it.
- Docs updated: the orch role contract (when to set it in a board entry), `docs/overview.md`, the `driftengine` package doc and the `lym drift` help text.

## To check by hand

- A real tapestry or darn run on a board entry whose `run_settings` carries `drift.scope` (each of `full`, `partial`, `off`) was not run; the tests drive Drift-Baseline and `LoadConfig` directly, including the task-local layer.
  Check that `lym loom status` shows Drift-Baseline done with no agent spawned and that `_lym/drift/scope.json` holds the given scope.
