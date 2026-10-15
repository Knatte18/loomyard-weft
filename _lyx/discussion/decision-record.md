# Decision record — reed-bindings-repin

## Goal

A deployed `lyx` binary re-pins reed's server-wide navigation bindings on an already-running tmux server at its first reed op, without waiting for a boot or an attach.

## Scope

- In: the root-table bindings and the follow jump line that `pinBindingsLocked` (`internal/reedengine/bindings.go`) pins, a server-global build stamp written beside them, and a re-pin check every op under reed's blocking op lock runs.
- Out: follow's default, which stays "unset reads on".
- Out: the global status-bar options `pinGeometryOptionsLocked` sets; they keep pinning at boot and attach only.
- Out: the watchdog's try-lock re-apply and the name-repair loop, which stay off the re-pin path.

## Decisions

- **The build stamp is a server-global user option.**
  `pinBindingsLocked` ends by setting `@lyx_bindings_build` with `set-option -g` to the running build's identity: its VCS commit time and its revision label.
  A binary with no VCS stamp unsets the option instead.
  Rationale: the bindings are server-wide, so their provenance belongs on the server, readable with one tmux round trip and gone with the server.
- **Every blocking reed op checks the stamp before its step.**
  `withOpLock`, which every reed CLI verb's engine call passes through, reads the stamp after taking the lock and calls `pinBindingsLocked` when the stamp is stale.
  A read that fails, as with no server running, skips the check and is logged at Debug.
  Rationale: one site covers strand add, window open, status and every other verb, and a fresh boot pins anyway.
- **A newer build wins, an older one never re-pins.**
  The stamp is stale when it is absent or unparseable, or when it names a different build whose commit time is not after the running build's.
  A running build with no VCS stamp never re-pins.
  Bound: a long-lived process of an older `lyx`, such as a run's driver started before the deploy, cannot pin its stale bindings back over a newer build's, and an unstamped test or `go run` binary leaves a running server alone; two builds of the same commit time, such as a dirty dev build and the clean build of its commit, can re-pin over each other.
- **The running identity is an Engine field.**
  `New` fills it from `buildvcs.Running()`, and package tests set it directly.
  Rationale: tests need a stamped identity, which a test binary never carries.

## Constraints

- `PATTERN-told-geometry`: the engine derives no path; the build identity is read from the binary, not from a path.
- `PATTERN-test-economy` and `PATTERN-test-speed`: the stale rule is a pure function with a table test, and the op-lock wiring is pinned against the scripted fake tmux.
- Each tmux failure on the re-pin path is non-fatal and logged, matching `pinBindingsLocked`'s existing contract.
- The `reedengine` package doc and `bindings.go`'s file header describe the re-pin in the same commit.

## Auto-mode assumptions

- `buildvcs.Identity.Time` is the commit time Go stamps from `vcs.time`, and a later production deploy carries a later commit time.
- A `display-message -p` read of a global user option answers the option's value through the session's inheritance, and empty when it is unset.

## Open risks

- A deploy that changes the global status-bar format without changing a binding still leaves the old status bar until the next attach.
- Two binaries of the same commit time, one dirty, alternate the bindings while both run reed ops.

## Acceptance criteria

- The stale rule: an absent or garbled stamp is stale; the running build's own stamp is not; an older build's stamp is stale; a newer build's stamp is not; an unstamped running build never finds a stamp stale.
- `pinBindingsLocked` sets the stamp to the running build's identity, and unsets it for an unstamped build.
- A reed op on a server whose stamp names an older build re-pins the bindings and the stamp before its step; one whose stamp names the running build issues no `bind-key`.
- A reed op whose stamp read fails issues no `bind-key` and still runs its step.

## Notes for the plan writer

- `pinBindingsLocked` and `bindingArgvs` live in `internal/reedengine/bindings.go`; `withOpLock` in `internal/reedengine/lock.go`.
- `internal/buildvcs` reads the running binary's VCS stamp.
- `faketmux_test.go` is the scripted tmux fake; an unscripted verb answers empty.
