# Card 9 — batten-innerrun-seams

**Create:**
- `internal/battenshed#ChildApproval`

**Edit:**
- `internal/battenshed#InnerRunDeps`
- `internal/battencli#battenCLI.wire`
- `internal/shedrecipe#innerRunEntry`
- `internal/battencli/wire_test.go`
- `internal/battencli/run_test.go`
- `internal/shedrecipe/entries_batten_test.go`
- `internal/shedrecipe/fixture_test.go`
- `internal/shedbuild/fixture_test.go`
- `internal/battenrecipe/fixture_test.go`

**Uses:**
- `internal/loomengine#LoomDriverStrandName`
- `internal/reedengine#ErrNoSession`
- `internal/landingshed#ReadApproval`
- `internal/reedengine#Engine.Status`

**Intent:**
Add the two injected seams `Run-Shed` needs to resume an approved child and to wait for its driver, and wire them from prime, before the producer uses them (card 10).
battenshed gains no loom, landing or resolver import (its seam-enforcement scan).

In `internal/battenshed/deps.go`:
- `ChildApproval` carries `ApprovedAt` and `HeadSHA` strings: the identity of one operator approval, as battenshed's own type.
- `InnerRunDeps` gains `ReadApproval func() (ChildApproval, bool, error)` (found is false when no approval record exists), `DriverAlive func(ctx context.Context) (bool, error)` (whether the child's driver strand is live), and `Now func() time.Time` (the clock the driver-exit grace reads; nil resolves to `time.Now` in `NewInnerRun`, the same way a nil `Sleep` resolves to `waitOrCancel`).
  Each field's doc comment says when `Call` invokes it and that it is resolved on `Call`, never at wiring time.

In `battenCLI.wire`, fill both seams, resolving the task worktree lazily inside each closure as every other seam there does:
- `ReadApproval` resolves the task worktree's location and decodes `loomengine.LoomApprovalPath(taskLocation)` through `landingshed.ReadApproval`, mapping `landingshed.Approval` onto `battenshed.ChildApproval`.
- `DriverAlive` answers false when `taskWorktreePresent` says the task worktree is gone; otherwise it builds the reed engine over the task worktree exactly as `Teardown.Shutdown` does (`reedengine.LoadConfig` on `AnchorPath()`, `hubgeom.ReedGeometry`), reads `Status()`, and reports whether a strand named `loomengine.LoomDriverStrandName` is `Live`.
  `Status()` never boots a session, and on an absent one it returns an error wrapping `reedengine.ErrNoSession` (card 4): the closure answers false for that error, since no session means no driver, and returns every other `Status()` error unchanged.
  Factor the closure's body into a helper over an injected status reader (`driverAliveFrom(present bool, status func() (reedengine.StatusResult, error)) (bool, error)`), so its answers are unit-testable without tmux.

`innerRunEntry` adds `requireSeam` checks for `InnerRun.ReadApproval` and `InnerRun.DriverAlive`, and not for `InnerRun.Now`, whose nil value selects the production clock; update its doc comment.

Tests: `wire_test.go`'s lazy-seam test covers the two new closures (non-nil after `wire`, and neither touches the absent task worktree at wiring time), and a table over `driverAliveFrom` covers its answers: task worktree absent is false without reading status; a status error wrapping `reedengine.ErrNoSession` is false with no error; any other status error is returned; a `loom-driver` strand that is `Live` is true; one that is not live, or no such strand, is false.
`entries_batten_test.go`'s missing-seam table gains `InnerRun.ReadApproval` and `InnerRun.DriverAlive` rows, and a case asserts a nil `InnerRun.Now` is accepted.
Every test fixture that builds an `InnerRunDeps` literal for a recipe build (the shedrecipe, shedbuild and battenrecipe fixtures, and battencli's `run_test.go`) fills both new seams with stubs (no approval, driver not live), so the new `requireSeam` checks do not refuse them.

**ImpactSummary:** Three new optional-at-type-level fields on InnerRunDeps, filled by battencli and required by the InnerRun registry entry; the producer does not read them yet.
