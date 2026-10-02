# Card 8 — loom-reviewer-resolved

**Edit:**
- `internal/loomcli/wiring.go`
- `internal/loomcli/parentreview_wiring_test.go`
- `internal/loomcli/parentreview_integration_test.go`
- `internal/loomcli/smoke_parentreview_test.go`

**Uses:**
- `plan:internal/hubgeom#ResolveParent`
- `plan:internal/hubgeom#Parent`
- `internal/hubgeom#ReedGeometry`
- `internal/reedengine#Engine.Status`
- `internal/reedengine#ErrNoSession`
- `internal/lyxcwd#ResolveWorktree`
- `internal/fabricengine#WorktreePath`

**Intent:**
Resolve the parent-review reviewer at use instead of reading it from the seed, and tell the gate whether that reviewer is live (Decisions 3 and 4).

`newParentReviewConfig` calls `hubgeom.ResolveParent(location)` in place of `shedrun.ReadSeed`; `Reviewer` and the delivery prompt's reviewer name are `Parent.Name`.
A resolver error is a wiring error, as a seed read error is today.
It takes the already-loaded reed config (`reedCfg` in `wire`) as a new argument and sets `ReviewerLive`:

- `Parent.Worktree` empty (no parent, or a legacy seed name): nil, so the gate treats the reviewer as live;
- otherwise a closure that resolves the parent worktree with `lyxcwd.ResolveWorktree(fabricengine.WorktreePath(location, Parent.Worktree))`, builds its geometry with `hubgeom.ReedGeometry`, and reads `reedengine.New(reedCfg, geom).Status()`.
  An error wrapping `reedengine.ErrNoSession` answers not live with no error; any other error is returned for the gate to Warn on; otherwise the reviewer is live when a strand whose `Name` equals the reviewer is `Live`.

Factor the strand-list check into a pure helper over a `func() (reedengine.StatusResult, error)` so it is testable without tmux.
The closure resolves the parent worktree on each call, never at wiring time, so a parent started after the run is seen.
Update `newParentReviewConfig`'s doc comment.

Tests: in `parentreview_wiring_test.go`, replace `TestNewParentReviewConfig_ReviewerFromSeedParent` with cases over the resolver's answer (a legacy seed name gives that reviewer and a nil `ReviewerLive`; no parent gives an empty reviewer), and table-test the strand-list helper: a live strand with the reviewer's name is live, a dead one or a different name is not, `ErrNoSession` is not live with no error, another status error is returned.
In `parentreview_integration_test.go`, the hub fixture's pair is created from the prime, so assert the request's reviewer and the delivery prompt carry `<shortname>:orch` rather than the seeded legacy name, and drop the seeded `Parent` the test no longer needs.
Keep `smoke_parentreview_test.go` compiling and consistent with the resolved reviewer (it is compiled by `go vet -tags smoke`, not run).

**ImpactSummary:** Changes the source of the parent-review reviewer from the seed to the origin-record resolver and adds the liveness seam; no other loom wiring changes.
