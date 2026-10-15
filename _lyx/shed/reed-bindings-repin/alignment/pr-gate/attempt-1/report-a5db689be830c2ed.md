---
verdict: APPROVED
---

## Scope

The board entry asks that a new `lyx` binary re-pin reed's navigation bindings on a running tmux server without waiting for an attach, through any reed command that touches the server, with follow's default kept as it is.
The decision record has no `Added after Discussion` entries, so the board body is the measure.

The diff of PR 648 at `235e399c7` against `main` does that and no more:

- `pinBindingsLocked` stamps the server-global option `@lyx_bindings_build` with the build's VCS commit time and revision label, and unsets it for an unstamped binary (`internal/reedengine/bindings.go`).
- `withOpLock` calls `repinStaleBindingsLocked` under the lock ahead of every op's step, so strand add, window open, status and every other blocking verb re-pin a server another build pinned (`internal/reedengine/lock.go`).
- Follow's default is untouched; `follow.go` and `ownwindow.go` are not in the diff.
- `doc.go` and `bindings.go`'s header describe the re-pin in the same change, as CLAUDE.md requires.

The orch's recommendation re-pins on any different build; the run re-pins only when the pinned build's commit time is not after the running one's.
That narrowing keeps the entry's expected outcome, a new binary re-pins, and only stops an older long-lived process from pinning stale bindings back, so it is no scope deviation.

The two open risks carried over, a status-bar-only change waiting for the next attach and two same-commit-time builds alternating, name no unrecorded parent correction.

## Overlap

No parent commits are listed.

- `engine-registry` (row 12): changes `internal/reedengine/doc.go` at the name-repair paragraph and `internal/reedcli/watchdog.go`'s SessionNamer import; this run changes `doc.go` at the navigation-bindings paragraph. Text-only overlap, no design conflict. This run (row 22) lands first; `engine-registry` takes its changes at Merge-In.
- `creel-core` (row 12): changes `internal/reedengine/lifecycle.go`'s `StrandStatus`; this run does not touch that file, and the two share only the package. Text-only overlap. This run lands first.
- `sibling-overlap-check` (row 17) and `plan-test-file-path` (row 9): no shared code.

Shared bookkeeping: `cmd/lyx/testdata/test-budget.yaml` (`internal/reedengine` 268 to 269), also edited by `engine-registry`, `creel-core` and `sibling-overlap-check` on other rows.
