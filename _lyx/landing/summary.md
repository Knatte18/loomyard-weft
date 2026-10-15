# reed: re-pin stale navigation bindings at the first op of a new build

A deployed `lyx` now re-pins reed's root-table navigation bindings and the follow jump line on an already-running tmux server at its first reed op.
Before, `pinBindingsLocked` ran only at boot and in the attach pre-flight, so a server booted by an older build kept its old Alt+Up/Alt+Down bindings.
Those bindings do not write `@lyx_follow off`, and an unset `@lyx_follow` reads as on, so each new agent window pulled the viewer back.
The only fix was to run `lyx reed attach` once in any pair.

Closes #646

## Design

- `pinBindingsLocked` ends by writing a server-global option, `@lyx_bindings_build`, holding the pinning build's VCS commit time (RFC 3339 UTC) and revision label.
  A binary with no VCS stamp unsets the option instead.
- `withOpLock` reads the option after taking the op lock and before the op's step, and calls `pinBindingsLocked` when `bindingsStale` says so.
  One site covers every blocking reed verb: strand add, window open, status and the rest.
- `bindingsStale` is a pure function.
  An absent or unparseable stamp is stale, and the running build's own stamp is not.
  Another build's stamp is stale unless its commit time is after the running build's.
- A newer build wins.
  A long-lived process of an older `lyx`, such as a run's driver started before the deploy, cannot pin stale bindings back over a newer build's.
- A binary with no VCS stamp, such as a test binary or `go run`, never re-pins and leaves a running server alone.
- A failed stamp read, as with no server running, is logged at Debug, skips the check and does not fail the op.
- The running identity is an `Engine` field filled by `buildvcs.Running()` in `New`, so package tests can set a stamped identity directly.

Follow's default is unchanged: an unset `@lyx_follow` still reads as on.

## Known bounds

- Two builds with the same commit time, such as a dirty dev build and the clean build of its commit, can re-pin over each other while both run reed ops.
- The global status-bar options still refresh only at boot and attach, so a deploy that changes only the status-bar format shows the old bar until the next attach.
- The watchdog's try-lock re-apply and the name-repair loop do not re-pin.

## Tests and docs

- A table test covers the stale rule.
- Tests against the scripted fake tmux pin that `pinBindingsLocked` writes or unsets the stamp, that an op on a server stamped by an older build re-pins before its step, that a server stamped by the running build gets no `bind-key`, and that a failed read still runs the step.
- The `reedengine` package doc and the `bindings.go` header describe the re-pin, and `cmd/lyx/testdata/test-budget.yaml` is raised for the new tests.

## Check by hand

Not run: deploy a new `lyx` over a live reed server and confirm that Alt+Up/Alt+Down switch windows after the first reed op, without an attach.
