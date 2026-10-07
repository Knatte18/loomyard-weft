# Decision record — Prompt-cache TTL per agent role

## Goal

Every lyx agent runs as a Claude Code main session, so on a subscription each one writes its prompt cache at the 1-hour price (2× base input) even when it never idles past 5 minutes.
Shuttle sets the prompt-cache TTL per role when it starts or resumes a session: 1 hour for the loom driver and the Webster Master (Merriam), 5 minutes for every other role, configurable per role.

## Scope

In:

- Two new keys in `shuttle.yaml` and its embedded template: a default TTL and a per-role override map.
- `claudeengine` resolving the run's TTL from those keys and the spec's role, and putting `CLAUDE_CODE_PROMPT_CACHE_TTL` on both the launch line and the resume line.
- Validation of TTL values in `claudeengine`, and the resolved TTL in the spawn log.
- Docs: `shuttleengine`'s and `claudeengine`'s `doc.go`, and `docs/overview.md` where it lists the shuttle config keys or the env carried on the pane line.

Out:

- The subagent and fork bucket (`CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL`): lyx leaves it at Claude Code's default, which is 5 minutes.
- Any per-call TTL field on `shuttleengine.Spec`: no caller chooses a TTL.
- Changing any role name or any caller's spec.

## Decisions

### D1 — The env var on the pane line is the mechanism

Decision: claudeengine wraps the launch line and the resume line in `CLAUDE_CODE_PROMPT_CACHE_TTL=<ttl>` through `sh.WithEnv`, the way it already carries `CLAUDE_CODE_FORK_SUBAGENT` and `CLAUDE_ENV_FILE`.
Rationale: the reed server env is scrubbed of `CLAUDE_CODE_*` at boot, so the pane line is the only route; the env var outranks every Claude Code control but `FORCE_PROMPT_CACHING_5M`, so an operator's `promptCacheTtl` setting or `ENABLE_PROMPT_CACHING_1H` cannot override the role's TTL.
The resume line carries it too, because reed replays that line verbatim when it rebuilds a session and the resumed claude inherits nothing from the launch.

### D2 — Two flat config keys in `shuttle.yaml`

Decision: `shuttleengine.Config` gains `claude_prompt_cache_ttl` (string, template default `5m`) and `claude_prompt_cache_ttl_roles` (map from role to TTL, template default `{driver: 1h, webster: 1h}`).
Rationale: `claude_`-prefixed keys already live in `shuttleengine.Config` (`claude`, `claude_deny_agent_tool`), so this follows the existing split: shuttleengine carries the value, claudeengine interprets it.
A key absent from an existing `shuttle.yaml` resolves to its template default under `PATTERN-config-strictness`, so existing worktrees get the split without an edit.
A `claude_prompt_cache_ttl_roles` map present in the file replaces the template's map whole, never merges with it, so an operator can drop a role back to the default by leaving it out.

### D3 — The role key is `Spec.Role`, matched exactly

Decision: the TTL for a run is `claude_prompt_cache_ttl_roles[spec.Role]` when that entry exists, else `claude_prompt_cache_ttl`.
An empty `Spec.Role` takes the default.
Rationale: `Spec.Role` is the role segment every spawning module already sets from its own constant (`driver` in `loomcli`, `webster` as `MerriamStrandRole` in `websterengine`), so no caller changes and the policy lives in one table.
The map keys are strand role segments, not `NameOverride` values and not webster's internal `RoleMaster`.
Bound: a map key naming no real role matches nothing and its runs fall to the default; the spawn log (D5) shows the TTL each run got, which is how a typo is seen.

### D4 — claudeengine validates the values

Decision: claudeengine accepts exactly `5m` and `1h` for the default and for every map value, and refuses a run's start otherwise, with an error that names the offending key, the role when it is a map entry, the accepted values, and `shuttle.yaml` as the file to fix.
The check covers the whole configured set, not only the value the run resolves to, so one bad entry is found on the first spawn rather than on the first spawn of that role.
Rationale: `5m|1h` is Claude Code vocabulary, and under `PATTERN-shuttle-provider-seam` provider vocabulary is validated by the engine, as `Effort` and `PermissionMode` are; Claude Code itself ignores any other value silently, so without the check a typo would quietly fall back to Claude Code's default.

### D5 — The resolved TTL rides the spawn log

Decision: the existing spawn log line for a claude session (`PATTERN-spawn-observability`) records the resolved TTL and the role it was resolved for.
Rationale: it is the only place a run's TTL is visible without reading the pane command, and D3's bound relies on it.

### D6 — The orch keeps the default

Decision: the orch session (`orchStrandName`) is not in the template's 1-hour map and gets 5 minutes.
Rationale: the operator's decision on the board entry (2026-10-07) names the driver and the Webster Master for 1 hour and "every other role" for 5 minutes; the per-role map lets the operator add the orch without a code change.

## Constraints

- `PATTERN-shuttle-provider-seam`: the env var name and the `5m|1h` vocabulary live only in `claudeengine`; `shuttleengine` holds the two config fields and their template defaults, and never interprets the values.
- `PATTERN-config-strictness`: `shuttleengine` stays a degrading caller (`LoadOrTemplate`); missing keys resolve to template defaults and are logged.
- `PATTERN-shell-mechanics-seam`: the env assignment is built through `internal/shell` (`sh.WithEnv`), never by string concatenation.
- `PATTERN-spawn-observability`: the TTL goes on the existing spawn log line through `internal/logger`.
- `PATTERN-test-speed`, `PATTERN-test-economy`: the behavior is covered by untagged unit tests at claudeengine's and shuttleengine's surfaces, extending the existing launch-line, resume-line and config tests where they fit.
- `PATTERN-comment-line-breaks`: new Go comments use semantic line breaks.
- Claude Code v2.1.242 or later honors the variable; the installed version (2.1.292) does.
  An older Claude Code ignores it and runs at its own default, which is no regression.

## Auto-mode assumptions

- The orch keeps 5 minutes, per the board entry's literal "every other role" (D6).
- Webster's recovery session (`recovery` role) keeps 5 minutes: it is a short focused session, not a waiting one.
- lyx does not set the subagent-bucket TTL; Merriam's implementer forks stay at 5 minutes, which suits forks that run without idle gaps.
- Unknown role keys are accepted, not refused: `lyx reed add` and `lyx shuttle run` take caller-chosen roles, so the set of valid roles is open.

## Open risks

- The orch idles on the operator, often past 5 minutes, and its context is large; at 5 minutes each return after a pause rewrites the whole cache at 1.25×, which may cost more than 1 hour would.
  The operator can add `orch` to the map; whether to make that the default is the operator's call.
- The 508 s maximum gap measured for the driver is from one run (`landing`); producers that spawn shells or wait on gates may idle longer than the 90 s seen there, which would make 5 minutes a net loss for them.
  The spawn log and transcript usage (`ephemeral_5m_input_tokens` vs `ephemeral_1h_input_tokens`) let a later run check this.
- `FORCE_PROMPT_CACHING_5M=1` in the environment Claude Code sees overrides the 1-hour roles; lyx does not detect it.

## Acceptance criteria

- A driver run's launch line and resume line both carry `CLAUDE_CODE_PROMPT_CACHE_TTL=1h`; a Webster Master run's lines carry `1h`; a run of any other role, and a run with an empty role, carry `5m` — all under the template's defaults.
- A `claude_prompt_cache_ttl_roles` entry for a role changes that role's TTL, and a map present in the file without `driver` gives the driver the default.
- A changed `claude_prompt_cache_ttl` changes the TTL of every role without a map entry.
- A `shuttle.yaml` lacking both keys resolves to the template's defaults.
- A default or map value other than `5m` or `1h` refuses the run's start with an error naming the key, the role when it is a map entry, and the accepted values; no pane is spawned.
- The spawn log line records the resolved TTL and role.
- No `shuttleengine` (non-`claudeengine`) file names `CLAUDE_CODE_PROMPT_CACHE_TTL` or the `5m|1h` vocabulary beyond the template defaults; the existing seam enforcement test stays green.
- `go test ./...` passes.

## Notes for the plan writer

- The launch and resume line builders sit side by side in `internal/shuttleengine/claudeengine/command.go`, with `forkSubagentEnvKey` as the precedent for a `CLAUDE_CODE_*` variable that must ride the pane line.
- `internal/shuttleengine/template.yaml` and `config.go` hold the existing `claude_*` keys; the hub's live `_lyx/config/shuttle.yaml` is not part of this task's diff unless reconcile convention says otherwise.
- Role constants: `loomcli.driverRole` (`driver`), `websterengine.MerriamStrandRole` (`webster`), `orchcli`'s `orchStrandName`.
