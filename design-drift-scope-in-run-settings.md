# Drift scope settable in a task's run_settings; Drift-Baseline agent only when unset

`drift-scope-in-run-settings` · Task · feature, drift, config · running · Code-Burler-Review

Imported from [issue #30](https://github.com/Knatte18/loomyard/issues/30).

## Wanted

Let a task fix its drift scope in its board entry, through `run_settings`, so the operator decides it when the task is discussed:

```
"run_settings": {"drift": {"scope": "full"}}
"run_settings": {"drift": {"scope": "partial", "skip": ["<fixture>", "..."]}}
"run_settings": {"drift": {"scope": "off"}}
```

- When `drift.scope` is set, Drift-Baseline records it as given and spawns no agent.
- When it is not set, the Drift-Baseline agent decides as today.

## Why

The scope choice is a judgement the operator often makes already, while discussing the task, with knowledge the agent lacks.
In the Models hub the agent picked scope full for a change confined to test projects the suite cannot reach (#25).
It predicts drift from the README and test setup, which do not say what each baseline fixture runs; a trial showed that a 1% change in steel heat capacity drifts only the temperature fixtures and not the quasi-steady-state one, which the agent cannot know.
Today the only way to correct a wrong choice is to pause after Drift-Baseline, edit `_lym/drift/scope.json` in the task worktree, `lym fabric commit`, and resume, since `lym drift scope` refuses a second call.

The drift config has no scope key now (`lym config drift --resolved` shows `enabled`, the commands, the models and `verify_max_runs`), so `run_settings` cannot carry it.