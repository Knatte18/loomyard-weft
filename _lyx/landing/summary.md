# shuttle: set the prompt-cache TTL per agent role

Every lyx agent runs as a Claude Code main session, so on a subscription each one writes its prompt cache at the 1-hour price (2× base input), even when it never idles past 5 minutes.
Shuttle now sets `CLAUDE_CODE_PROMPT_CACHE_TTL` per role when it starts a session: 1 hour for the loom driver and the Webster Master, 5 minutes for every other role.
The split is configurable per role in `shuttle.yaml`.

## What changes

- `shuttle.yaml` and its template gain `claude_prompt_cache_ttl` (default `5m`) and `claude_prompt_cache_ttl_roles` (default `{driver: 1h, webster: 1h}`).
  A key absent from an existing `shuttle.yaml` resolves to the template default, so existing worktrees get the split without an edit.
- `claudeengine.Prepare` resolves the TTL from `Spec.Role` and the config, and wraps both the launch line and the resume line in the env var through `sh.WithEnv`, as it already does for `CLAUDE_CODE_FORK_SUBAGENT`.
- `Prepare` logs one `Info` line with the role and the resolved TTL.
- `shuttleengine/doc.go`, `claudeengine/doc.go` and `docs/overview.md` describe the keys.

## Design decisions

- **Pane line, both lines.**
  The reed server env is scrubbed of `CLAUDE_CODE_*` at boot, so the pane line is the only route.
  The variable outranks an operator's `promptCacheTtl` setting and `ENABLE_PROMPT_CACHING_1H`.
  Reed replays the resume line verbatim when it rebuilds a session, so it carries the TTL too; both lines take one value resolved once in `Prepare`.
  A rebuilt or resumed session keeps the TTL its run started with, and a later `shuttle.yaml` edit reaches only runs started afterwards.
- **Open map.**
  A `claude_prompt_cache_ttl_roles` map present in the file replaces the template's whole, so an operator drops a role back to the default by leaving it out.
  That needs the key declared an open map: `shuttleengine.ConfigOpenMaps()` feeds `LoadOrTemplate`, and the `shuttle` entry in `configreg` carries it as `OpenMaps`.
  Without it, the loader re-adds the template's `driver` and `webster` entries, and `lyx config reconcile` prunes an operator's own entry.
- **Role key.**
  The map is keyed by `Spec.Role`, matched exactly; an empty role takes the default.
  No caller or role name changes.
  A key naming no real role matches nothing, and the spawn log shows which TTL each run got.
- **Validation in `claudeengine`.**
  Only `5m` and `1h` are accepted, for the default and every map value; anything else (empty, null, `1H`, numeric) refuses the run's start before any artifact is written or pane created.
  The error names the key, the role for a map entry, the accepted values and `shuttle.yaml`.
  The whole configured set is checked, so one bad entry refuses the next spawn of every role, mid-loom included, until the file is fixed; a started or rebuilt session is never affected.
  The env var name and the `5m|1h` vocabulary stay in `claudeengine`; `shuttleengine` only carries the two fields and their template defaults (`PATTERN-shuttle-provider-seam`).
- **Orch keeps 5 minutes.**
  The ticket names only the driver and the Webster Master for 1 hour.
  Adding `orch` to the map needs no code change.
- **Not touched.**
  The subagent and fork bucket (`CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL`) stays at Claude Code's default of 5 minutes, and `Spec` gains no per-call TTL field.

## Test changes

- Fixtures that hand `Prepare` a `Config` now set both TTL fields, since an empty value is refused.
- `wait_test.go` no longer compares `Config{}` by equality, and `envkit` lists the new map as nil-legal.
- `cmd/lyx/testdata/test-budget.yaml` raises the `shuttleengine` and `claudeengine` budgets by one test each.

## To check

- Claude Code v2.1.242 or later honors the variable; an older one ignores it and runs at its own default.
- `FORCE_PROMPT_CACHING_5M=1` in the environment Claude Code sees overrides the 1-hour roles, and lyx does not detect it.
- The orch idles on the operator, often past 5 minutes, and its context is large; at 5 minutes each return after a pause may cost more than 1 hour would.
  Compare `ephemeral_5m_input_tokens` and `ephemeral_1h_input_tokens` in transcript usage, and decide whether `orch` joins the default map.
- The 508 s maximum driver gap was measured on one run (`landing`); producers that wait on gates may idle longer than the 90 s seen there, which would make 5 minutes a net loss for them.
- Not run by hand: a live session started under each TTL, checked against the pane command and the spawn log line.
