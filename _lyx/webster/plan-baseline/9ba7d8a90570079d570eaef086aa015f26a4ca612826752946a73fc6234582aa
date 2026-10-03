# Card 2 — remove-finishes-half-removed-pair

**Edit:**
- `internal/fabricengine#Topology.Remove`
- `internal/fabricengine#RemoveResult`
- `plan:internal/fabricengine#Topology.RemoveRefusal`

**Create:**
- `plan:internal/fabricengine#ErrPairNotFound` -> `var ErrPairNotFound error`
- `internal/fabricengine/remove_finish_integration_test.go`

**Uses:**
- `plan:internal/fabricengine#commitPendingRecords`
- `internal/fabricengine#parseWarpSHATrailer`
- `internal/fabricengine#removeWeftWorktree`
- `internal/fabricengine#deleteWarpBranch`
- `internal/fabricengine#deleteWeftBranchOnRemote`
- `internal/fabricengine#Topology.RemovePairBranch`

**Intent:**
Implements D5: `Remove` finishes a half-removed pair instead of answering "worktree not found".

When the task worktree is absent, `RemoveRefusal` and `Remove` skip only the checks that need it (warp-side dirtiness, the task side of the merge probe) and keep every other refusal.
`Remove` then performs whatever teardown remains, in the normal order, through the same gated helpers the full path uses:
pending-record commit, archive, portal and launchers, sibling worktree, both branches locally, and with `--remote` the sibling branch on origin.
Card 3 adds the task branch on origin.

- Pending-record commit: when the sibling worktree is present, `commitPendingRecords` runs as on the full path.
  Its `Warp-SHA` trailer uses the local task branch tip when that branch exists, else the last `Warp-SHA` trailer recorded on the sibling branch (`parseWarpSHATrailer` over its log).
  With neither, the commit carries no trailer.
- A sibling branch present only on origin is archived from origin's copy (archiveWeftTip already falls back) and, with `--remote`, deleted there through `deleteWeftBranchOnRemote`.
  This is the case `RemovePairBranch` handled; that function is deleted by card 12.
  Without `--remote`, an origin-only copy stays and the result says so.
- The recorded parent comes from the origin record while the sibling worktree is present; with it gone the parent is empty and the task-branch gate falls back to its stricter reachability check alone.
- A path at the task worktree location that is not a registered linked worktree is reported in a new `RemoveResult` field (`StrayPath`, json `stray_path`) and never deleted; the rest of the teardown proceeds.

When nothing of the pair remains (no worktree on either side, neither branch locally, neither branch on origin when an origin is configured, no portal and no launcher entry), `Remove` returns an error wrapping the new sentinel `ErrPairNotFound`.
Its text contains "not found", which `removeHostileCases`' LeadingDash expectation relies on, and names a stray path at the pair location when one is present.
The sentinel is worded without naming either side of the pair, so `internal/battencli` can match it with `errors.Is`.

`RemoveResult` gains `Steps []string` (json `steps`, always an array): the steps this call performed, from a closed set of constants declared beside the type (record commit, archive, portal, launchers, task worktree, sibling worktree, sibling branch, task branch, sibling branch on origin), appended only after the step changed state.
It also gains `Finished bool` (json `finished`), true when the task worktree was already gone at entry.

Bound: the finish path deletes only what the full path would have deleted, through the same destruction gates in `destroy.go`; it adds no deletion target, and the archive still precedes every branch deletion.

Tests in `remove_finish_integration_test.go`, each converging on one re-run of `Remove`:
- Task worktree removed by hand, sibling worktree present and dirty under the pathspec: the record is committed and contained in the archive tag, and the sibling worktree and both branches are gone.
- Both worktrees gone, both branches left locally: both branches are deleted, the sibling branch only after its archive tag is pushed.
- Both gone, sibling branch only on origin, `--remote`: archived from origin and deleted there; without `--remote` it is kept and reported.
- A plain directory at the pair path beside a leftover branch: it is reported in `stray_path`, left on disk, and the branch teardown completes.
- Nothing left: `errors.Is(err, ErrPairNotFound)`, and with only a stray directory left the error text names it.

**ImpactSummary:** Remove's absent-worktree branch changes from a refusal to a finish path; RemoveResult gains three fields; no caller signature changes.

**Commit:** `2: fabric remove finishes a half-removed pair`
