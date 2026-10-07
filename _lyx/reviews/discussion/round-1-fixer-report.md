# Round 1 fixer report

Verdict was BLOCKING; every finding is fixed in `decision-record.md` or `support-log.md`.

## Changed

- **F1 (BLOCKING):** D2 now declares `claude_prompt_cache_ttl_roles` an open map: a `ConfigOpenMaps()` in `shuttleengine`, passed by `LoadConfig`, registered as `OpenMaps` on the `shuttle` entry in `configreg`.
  Scope's first bullet names it, an acceptance row pins the carried-whole behavior, and a plan-writer note points at the `batcher` precedent.
  `support-log.md` Q2 records the dependency.
- **F2:** The Goal now says the TTL is set when a session starts, on both lines.
  D1 states that `claudeengine.Prepare` resolves it once from `spec.Role` and `cfg`, that both builders get the same value, and that a rebuilt session keeps the TTL its run started with.
- **F3:** D4 gained a Bound: the refusal is in `Prepare` before any pane or run directory exists, it never touches a started or rebuilt session, and a bad entry for any role refuses every role's next spawn until `shuttle.yaml` is edited.
- **F4:** D5 now has `claudeengine.Prepare` log one `Info` line with role and TTL, with the reason it is not the `shuttleengine` "run started" line.
  Scope, Constraints, the acceptance criterion and Q5 follow; the criterion states the test captures the logger and runs without `t.Parallel`.
- **F5:** The D4 acceptance criterion names the refused empty, null, differently cased and numeric values, and the bad entry for a role other than the one starting.
- **F6:** The open-role-set premise cites `lyx shuttle run --role` only, in the Auto-mode assumptions and the support log's rejected alternative.

## Deferred

None.
