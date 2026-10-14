## Changes

- F1: `handOver` in `internal/reedcli/attach_unix.go` now resolves the tmux name with `exec.LookPath` and passes the resolved path as both the path and argv[0] of `syscall.Exec`.
  A failed lookup returns exit code 1 and the error.
  The `testing.Testing()` branch is unchanged and still runs `runAttachChild` with the original name.
  The doc comment names the PATH lookup.
  `lyx gate test --tags tmux ./internal/reedcli` passes.

## Disputed

None.

## Deferred

None.
