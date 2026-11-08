# Fixer-report — code review round 1

## Changes

- F1 fixed as proposed: `pattern/PATTERN-lym-files.md:25` now reads "A module's machine override sits under …". Commit `f68b0c8f`.
- F2 fixed as proposed: `docs/overview.md:513` now names codex's `sandbox_mode` beside gate's keys and drops "hub-wide". Commit `696633c2`.
- F3 fixed as proposed: `TestOpenLaunch` in `internal/shuttleengine/codexengine/open_test.go` has a `FullAccessSandboxMode` row.
  Its worktree layer writes `sandbox_mode: danger-full-access`, and the expected line is built with `sandboxFullAccess`.
  A `wantSandboxMode` field defaults to `workspace-write` for the other rows. Commit `95e10688`.
- Self-check finding fixed as proposed: the `--layer` flag help in `internal/configcli/configcli.go:674` no longer calls the machine override a hub-wide module's.
  Commit is the one after `95e10688`, titled "configcli: fix self-check …".

`lym gate test` passed for `./internal/shuttleengine/codexengine` and `./internal/configcli`.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: LOW, `--layer` flag help still says "hub-wide module's"; fixed as proposed.

Every fork returned a report.

## Disputed

None.

## Deferred

None.
