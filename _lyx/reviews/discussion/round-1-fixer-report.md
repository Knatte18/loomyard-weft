# Fixer report — discussion, round 1

## Changes

- F1 (BLOCKING), fixed as proposed: the premise holds, because `configengine.Layers` runs `ValidateLocal` on the local file, and that refuses a key the template lacks (`internal/configengine/layers.go:171`), while `hubreconcile.Ensure` has no caller in `shuttlecli`, `reedcli` or `pairteardown`.
  D4 now has `shuttle`'s `ConfigSpec` declare the five moved source keys as retired keys, through a new `configengine.Spec` field.
  `ValidateLocal` admits a retired key, and the loader drops it from the merged document.
  A retired key is never read.
  It is admitted only where it was before, so the TTL keys at the local layer, while the Executable and Guard keys stay refused there.
  The D4 Rationale says why: this keeps D2's guarantee across the deploy.
  Open risks names the local-layer case.
  There is a new acceptance row: before any reconcile, a pair with `claude_prompt_cache_ttl_roles` in `shuttle.local.yaml` loads `shuttle.yaml`, and stop and strand removal work.
  Two auto-mode assumptions were added or updated.
  The support log gains a rejected alternative (keep the refusal) and an updated Q4.
- F2, fixed as proposed: the Open risks bullet now names the python switch.
  Until the walk, an unreconciled `claude_deny_python: false` is ignored and `deny_python` resolves to `true`.
- F3, fixed as proposed: D4 now limits the template-default overwrite to base layers.
  At a sparse layer, any target value is kept as a kept-target collision.
  An overwritten template-default target is now reported.
  The Rationale and the auto-mode assumption match.
  The acceptance row adds a worktree `claude.yaml` with a template-default value, which is kept.
- F4, fixed as proposed: the acceptance row now expects `shuttle: {claude_deny_python: …}` to be refused as a `RunSettingsError` whose cause names `deny_python`.
  D3 says the renamed key is refused on run settings like every Guard key.
  The premise checks out: `internal/configreg/runsettings.go` runs `Migrate` and then `ValidateLocal`.
- F5, fixed as proposed: the D8 cycle-exception sentence is deleted.
  `go list -deps ./internal/shuttleengines` contains neither `hubgeom` nor `pairteardown`.
  The Rationale now limits the compile-error claim to an omitted argument and makes `reedengine.New` panic on an empty set.
  The D8 acceptance row adds that construction with an empty set fails.
- F6, fixed as proposed: the rejected-alternative clause is cut from D5's Rationale.

The support log's Review rounds section records round 1.

## Disputed

None.

## Deferred

None.
