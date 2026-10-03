# Card 13 — pair-teardown-invariant-and-docs

**Prosa:**
- `CONSTRAINTS.md`
- `docs/overview.md`
- `internal/fabricengine/doc.go`
- `internal/reedengine/doc.go`
- `internal/battenshed/doc.go`

**Create:**
- `manifest/designs/pair-teardown.md`
- `internal/pairteardown/teardown_enforcement_test.go`

**Intent:**
Lands the cross-cutting rule, its tripwire and the module docs the decision record lists, all with semantic line breaks.

`CONSTRAINTS.md`:
- A new **Pair Teardown Invariant** section, placed after the Batten Bookend Invariant: every teardown of a pair goes through `internal/pairteardown`, which ends the pair's reed session before any worktree is removed.
  No production package outside `internal/pairteardown` and `internal/fabricengine` calls `Topology.Remove`.
  The section also states the bounded quiet wait, that `--force` never answers it, that the package stays inside the Fabric Vocabulary Invariant, and that enforcement is `internal/pairteardown/teardown_enforcement_test.go` (a tripwire, not a completeness proof).
- The Fabric Git Invariant's teardown bullet drops `RemovePairBranch`.
  It gains the pending-record commit: `Remove` commits the sibling worktree's uncommitted changes under the scoped record pathspec before the archive tag, so the tag holds them, and `force` skips neither.

`docs/overview.md`: the module tree gains `internal/pairteardown/` with a one-line description.

`internal/fabricengine/doc.go`:
- One sentence (D10): `main-weft` does not advance on landing by design, since weft content is per-branch and never a merge participant and a landed pair's records live in its archive tag.
- The `Remove` and `Cleanup` descriptions cover the finish path, the pending-record commit, the remote task-branch deletion and `CleanupRemoteWarp`.

`internal/reedengine/doc.go`: the retiring mark as reed's own lifecycle bookkeeping, and `EndSessionByName` beside `ReapSession` as the gone-worktree session end.

`internal/battenshed/doc.go`: Worktree-Teardown's two halves call the pair-teardown composite.
Session shutdown no longer skips a gone task worktree, and a not-found removal is the done post-condition.

`manifest/designs/pair-teardown.md`: the module doc for `internal/pairteardown`.
It covers why the package exists (the import cycle, one sequence for two callers), the sequence (quiet wait, refusal probe, session end, removal), the quiet rule including retiring and parked drivers, the two caller policies, the finish path it relies on, and the out-of-scope items from the decision record.

`teardown_enforcement_test.go` (untagged, `internal/testkit/scankit` over the repository's production `.go` files) fails on any call of a method named `Remove` with four arguments in a file importing `internal/fabricengine`, outside `internal/fabricengine` and `internal/pairteardown`.
It also fails on any remaining reference to `RemovePairBranch`.
A self-test feeds it one offending and one clean source snippet so the scan cannot pass vacuously.

**Commit:** `13: pair teardown invariant, its tripwire and module docs`
