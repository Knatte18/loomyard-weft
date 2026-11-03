# Drift scope set in a task's run_settings skips the Drift-Baseline agent

A task's regression scope can now be fixed in its board entry, so the operator decides it while discussing the task instead of leaving it to the Drift-Baseline agent:

```
"run_settings": {"drift": {"scope": "full"}}
"run_settings": {"drift": {"scope": "partial", "skip": ["<fixture>", "..."]}}
"run_settings": {"drift": {"scope": "off"}}
```

## Config

`drift.yaml` gains two keys, `scope` (default `""`) and `skip` (default `[]`).
Both are classified `CategoryNone`, so the local layer, which Seed-Child writes from a task's `run_settings`, may hold them.
`LoadConfig` refuses a scope other than empty, `off`, `partial` or `full`, a `partial` scope with no skipped test, and a skipped test under any other scope.
These refusals name the key and point at both the board entry's `run_settings` and `lym config drift`.
`Config.Skip` is set only under `partial`.

## Drift-Baseline

When `drift.scope` is set, `DriftBaseline` now does the following before its agent branch:

- records the scope with its `skip` list when no scope record exists yet;
- writes `baseline.md` itself, from the recorded scope, when the file is missing.

Both files then exist, so no agent is spawned.
The capture logic after that point is unchanged: scope `off` takes no capture, and `partial` and `full` start the capture.
A scope already recorded wins over the config value.
That keeps a parent's `lym drift scope` replacement at a blocked Drift-Baseline or Drift-Await, and an earlier pass's record, intact across a resume.
When `drift.scope` is empty, the agent decides the scope, as before.

The generated `baseline.md` names the scope and its skipped tests and says no agent predicted per-test drift.
Drift-Verify's agent reads this file as its expectation.

## Design decisions

- **Keys `scope` and `skip` at the top level of `drift.yaml`.**
  This matches the `run_settings` shape the board entry asks for, `{"drift": {"scope": "partial", "skip": [...]}}`.
  A hub can also set `scope` in its base layer, which then applies to every task with no `run_settings` override.
- **The config never overrides a scope that is already recorded.**
  The scope record stays the single source of truth once written.
- **The orch role now tells the orch to put a scope the operator decides into `run_settings`.**
  This is how a scope decided during discussion reaches the run.

## Docs

The `driftengine` package doc, the `lym drift` and `lym drift scope` help, the overview's regression-rows paragraph, and the drift template's key descriptions describe the new keys and behaviour.

## Tests

- `TestLoadConfig` gains rows for these cases:
  - `full` and `partial` scopes that load;
  - an unknown scope;
  - `partial` with no skipped test;
  - a skipped test under `full`;
  - a local-layer scope loading over the board's.
- `TestDriftBaseline` gains rows for these cases, each with no agent spawned:
  - a configured `full`;
  - a configured `off`;
  - a configured `partial`;
  - a recorded scope winning over the configured one.
- `TestModules_EveryTemplateKeyClassified` lists the two new local keys.
