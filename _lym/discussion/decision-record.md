## Goal

```markdown
## Seen (odd hub orch, 2026-11-18, by creel)

In hub `odd` (repository `Knatte18/oddments`), `deny_python: false` is set in the shuttle config at the default layer, committed in `_bolt`; `lym config --resolved shuttle` shows `deny_python: false # default`.
Yet every new orch session that `lym orch` launches, also after a full restart, still carries the python guard's notice in its system prompt, and the old session's hook still refused Python.
Hubs whose repositories develop Python cannot work.

The row hangs on `shellguard.SwitchDenyPython`, and `claudeengine`'s `shellSwitchInstalled` reads `cfg.DenyPython`; the `# default` mark suggests the resolved value comes from the template, not the hub's layer, so the launch path of `lym orch` may load the shuttle config from other dirs than `lym config` resolves.

## Idea

Find which config dirs the orch launch hands the shuttle config loader, make it read the hub's layers, and add a test: with `deny_python: false` at the default layer, an orch launch carries neither the notice nor the hook row.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-18): The python hook reads deny_python live

Decision: The python guard's hook reads the hub's current deny_python on each call instead of having the value baked in at launch, so flipping the flag takes effect in running sessions with no restart of the session or its watchers. The system prompt notice still follows the value at launch.

Rationale: Operator, 2026-11-18: turning Python on must not need a restart of every session and watcher.

### Added after Discussion (parent, 2026-11-18): Accept round 1 of Code-Review with its fixes

Decision: Round 1's ten findings are all fixed; F2 is accepted as fixed differently: the verb falls back to the standalone state dir when no hub resolves, with no change to hook commands or engine signatures. No second review round.

Rationale: Every finding has a concrete fix and the gate passed; F2's fallback keeps the change inside the verb, which is the smaller of the two fixes the review offered. A second round costs more than the remaining risk.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
