# reed: re-pin stale navigation bindings on a running server

Fixes #646.

reed pinned its root-table navigation bindings (Alt+Up/Down/Left/Right, Alt+z and the status-bar click) only at server boot and in the attach pre-flight.
A deploy therefore left a running tmux server holding the previous build's bindings until someone ran `lyx reed attach`.
After the follow change landed, the old Alt+Up/Down bindings did not write `@lyx_follow off`, so each new agent window pulled the viewer back.

## What changes

- `pinBindingsLocked` (`internal/reedengine/bindings.go`) now stamps the server-global user option `@lyx_bindings_build` with the pinning build's VCS commit time (RFC 3339 UTC) and revision label, ahead of the `bind-key` calls.
  A binary with no VCS stamp unsets the option.
- `withOpLock` (`internal/reedengine/lock.go`) calls the new `repinStaleBindingsLocked` after taking the lock and before the op's step.
  It reads the stamp with one `display-message` and calls `pinBindingsLocked` when `bindingsStale` reports the stamp stale.
  Every reed verb that goes through the blocking op lock is covered, including strand add, window open and status.
- `bindingsStale` is the pure rule.
  An absent or unparseable stamp is stale.
  The running build's own stamp is not stale, and neither is the stamp of a build with a later commit time.
  Any other build's stamp is stale.
  A running binary with no VCS stamp never re-pins and never reads the stamp.
- The `Engine` carries its build identity as a field, filled from `buildvcs.Running()` in `New`.

Follow's default is unchanged: an unset `@lyx_follow` still reads on.

## Design decisions

- **A newer build wins.** Compared to re-pinning on any difference, ordering builds by commit time stops a long-lived process of an older `lyx` from restoring its stale bindings over a newer build's. A run's driver started before the deploy is one such process.
- **One site, the op lock.** All blocking reed ops pass through `withOpLock`, so no verb needs its own call. The watchdog's try-lock re-apply and the name-repair loop run on a tick and stay off this path.
- **A failed read is quiet.** With no server up, the stamp read fails; it is logged at Debug and the op runs unchanged. A fresh boot pins the bindings anyway.
- **Bindings only.** The global status-bar options that `pinGeometryOptionsLocked` sets still refresh only at boot and attach.

## Risks

- A deploy that changes only the global status-bar format still needs an attach to show it.
- Two builds with the same commit time, such as a dirty dev build and the clean build of its commit, re-pin over each other while both run reed ops.

## Tests

- `TestBindingsBuildStamp` (`internal/reedengine/bindings_test.go`) covers three things: the stale rule as a table, the stamp the pin writes for a stamped and an unstamped build, and `withOpLock` against the scripted fake tmux. The fake-tmux rows check that an older or absent stamp re-pins before the step, and that the running build's stamp, a newer build's stamp, a failed read and an unstamped binary issue no `bind-key`.
- The existing `set-option` counts in `TestPinGeometryOptionsLocked` and `TestAttachArgv_PreflightOnAKnownGoodSession` now include the stamp unset.
- `internal/reedengine`'s row in `cmd/lyx/testdata/test-budget.yaml` rises by one for the new top-level test.

The package doc (`internal/reedengine/doc.go`) and `bindings.go`'s file header now describe the stamp and the re-pin.
