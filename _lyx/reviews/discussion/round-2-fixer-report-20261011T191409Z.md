# Fixer report — discussion round 2

## Fixed

- F1 (BLOCKING, design): premise verified in `internal/reedengine/lifecycle.go` (`pinBootOptionsLocked` runs `set-option -g mouse`; `validateBootConfig` turns `debug_log` into tmux's global flags on the boot that spawns the shared server).
  `decision-record.md` D4: reed's `LocalDenied` adds `mouse` and `debug_log`; the reason sentence names an option the hub's shared tmux server takes at boot; the categories are stated as a test every key, including one the plan touches or adds later, takes; the `Bound:` line names a shared process's boot option.
  The `PATTERN-config-layers` Constraints line and the `LocalDenied` acceptance criterion (now naming `reed.mouse`) follow.
  The proposed "state that the denylist was derived by checking every open key" is written as a standing rule rather than as process history.
- F2 (MEDIUM, design): premise verified in `internal/shuttleengine/template.yaml` (`claude_prompt_cache_ttl_roles` replaces the template's whole map when present) and D1's open-map rule.
  D6 states an open map is compared and stripped as one unit and that the strip never changes a worktree's resolution; a new acceptance criterion pins equal resolution before and after, including an open map differing in one entry.
- F3 (LOW, design): premise verified in the reed and shuttle templates (`${env:LYX_REED_*}`, `${env:LYX_SHUTTLE_*}`) and `internal/yamlengine/resolve.go`.
  D4's rationale now rests on persistence: the denied keys persist in a pair only through a tracked file, an env marker binds only the process that sets the variable.
- `support-log.md`: round 2 entry under `## Review rounds`; Q5 names the added category.

## Disputed

None.

## Deferred

None.
