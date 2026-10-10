# Plan-Review round 3 — fixer report

I checked every finding against the code: `internal/pairteardown/teardown.go`, `internal/reedengine/lifecycle.go`, `internal/websterengine/cardverify.go` and its test, `internal/verifyrun/run.go`, and the `fabriccli` and `battencli` test mains.
Every premise holds, and all four are fixed.
`lyx loom validate-plan` reports `ok: true` after the edits.

## Fixed

- **F1** (BLOCKING, design), `10-tmuxkit-socket-check.md`.
  `probeQuiet` calls `Engine.Status`, and `requireSessionLocked` runs `has-session`.
  `endReedSession` calls `Engine.Down`.
  Both `fabriccli` and `battencli` run through `tmuxkit.Main`.
  So the directory rule would have failed their `integration` tier even though no server starts.
  The per-user directory is never a finding by itself now, and the check counts socket files only.
  The card states the bound for every tier: a server reed tears down itself within the run removes its socket file and escapes the check.
  The `PATTERN-test-isolation.md` sentence and the `TestCheckSockets` row change to match: the directory with no socket is clean in every tier.
- **F2** (MEDIUM, scope), `13-gatecli-prebuild.md`.
  `internal/websterengine#runCardVerify` and `cardverify_test.go` join `Edit:`, and `internal/websterengine/doc.go` joins `Prosa:`.
  The rerun strips the variable on both paths: `StripPrebuilt(lease.Env(os.Environ()))` slotted, `StripPrebuilt(os.Environ())` unslotted.
  `verifyrun.Run` has no other caller, so the bound now names both verify paths and holds.
  `TestRerunCardVerifies_AllPass` is already not parallel.
  It gains `t.Setenv` and a card running `test -z "$LYX_PREBUILT_LYX"`.
  The `ImpactSummary` follows.
- **F3** (MEDIUM, decision), `05-tmuxkit-hermetic-server.md`.
  I raised it to `ly:orch`, which chose to accept the bound and recorded the parent amendment "D4 bound: a server reed boots itself after a test's own down reads the operator config".
  Card 5's bound sentence cites that amendment by title, and so does the "Hermetic servers" section it adds to `PATTERN-test-isolation.md`.
  There is no tmux PATH wrapper.
- **F4** (LOW, design), `31-tierpurity-tmux-constraint.md`.
  I took a different fix than the review proposed.
  With `tmux` and `llm` held false, the helper evaluates the constraint under every assignment of its other tags, and the file is tmux-tier only when every evaluation is false.
  That needs no fixed GOOS list and no host GOOS.
  The row `integration && windows` (not tmux-tier) is added.

## Disputed

None.

## Deferred

None.
