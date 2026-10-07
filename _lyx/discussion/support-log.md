# Support log — cache-ttl-per-role

## Interview

The session ran autonomously; no operator answered.
Every question below was settled by an auto-pick, recorded in the question ledger.

1. Read the board entry: operator decision of 2026-10-07 fixes 1 hour for the driver and the Webster Master, 5 minutes for every other role, configurable per role, via `CLAUDE_CODE_PROMPT_CACHE_TTL`.
   Related entries read: `burler-cost` (a model per round, a split reviewer and fixer) and `two-agent-producer`; neither changes role names this task keys on, and a future split Burler role falls to the default unless the map names it.
2. Read Claude Code's prompt-caching doc: the env var takes `5m` or `1h` and ignores anything else; it outranks the `promptCacheTtl` setting and `ENABLE_PROMPT_CACHING_1H`, losing only to `FORCE_PROMPT_CACHING_5M`; forks and subagents sit in a separate bucket defaulting to 5 minutes.
3. Read `claudeengine/command.go`: launch and resume lines are built separately and both carry `CLAUDE_CODE_FORK_SUBAGENT` through `sh.WithEnv` because reed scrubs `CLAUDE_CODE_*` from its server env; the TTL takes the same route.
4. Read role constants across spawning modules: `Spec.Role` is set by every caller from its own constant; the driver is `driver`, Merriam is `webster`, the orch is `orchStrandName`.
5. Read `shuttleengine.Config` and `PATTERN-config-strictness`: `claude_*` keys already live in shuttleengine's config; missing keys resolve to template defaults.
6. Read `PATTERN-shuttle-provider-seam`: the env var name and value vocabulary belong in claudeengine.

## Rejected alternatives

- **A `CacheTTL` field on `shuttleengine.Spec`, set by each caller.** Spreads one policy across every spawning module and needs a config read in each; a role-keyed table in one config file is one place to change.
- **`promptCacheTtl` in the per-run `settings.json` shuttle writes.** Works on resume too, but an env var in the operator's environment would outrank it; the pane-line env var outranks every control except the force switch.
- **A nested `claude_prompt_cache_ttl: {default, roles}` key.** Two flat keys follow the existing `claude_*` split and keep the default a plain scalar a missing key resolves exactly; the roles map needs the open-map declaration either way.
- **Validating TTL values in `shuttleengine.LoadConfig`.** Earlier failure, but it puts Claude vocabulary into shuttleengine against `PATTERN-shuttle-provider-seam`; the engine validates, as it does for effort and permission mode.
- **Refusing unknown role keys in the map.** The role set is open (`lyx shuttle run --role` takes a caller role), so there is no closed list to check against; the spawn log makes a mismatch visible instead.
- **1 hour for the orch.** Argued by its long operator idle times, but the operator's decision names only the driver and the Master; left as an open risk and a one-line config change.
- **Setting the subagent bucket TTL as well.** Claude Code already gives forks 5 minutes; nothing asks for another value (YAGNI).

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): mechanism — env var on the pane line vs per-run settings vs Spec field. Pick: env var on launch and resume lines; matches the board's shape and outranks other controls.
- Q2 (resolved, auto-pick): config shape. Pick: two flat keys `claude_prompt_cache_ttl` and `claude_prompt_cache_ttl_roles`; a present map replaces the template's whole, which needs the map declared an open map.
- Q3 (resolved, auto-pick): which identifier keys the map. Pick: `Spec.Role`, exact match; empty role takes the default.
- Q4 (resolved, auto-pick): where values are validated. Pick: claudeengine at start, the whole configured set, refusing with key, role and accepted values.
- Q5 (resolved, auto-pick): unknown role keys. Pick: accepted; the log line from `claudeengine` records the resolved TTL and role.
- Q6 (resolved, auto-pick): the orch's TTL. Pick: default 5 minutes, per the operator's "every other role"; recorded as an open risk.
- Q7 (resolved, auto-pick): Webster's recovery session. Pick: default 5 minutes.
- Q8 (resolved, auto-pick): subagent and fork bucket. Pick: not set; Claude Code's 5-minute default stands.
