---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/hubreconcile/walk.go:209
    summary: reconcileHubWide pushes the board on every run, so a board with no reachable push destination never gets a build stamp and every start verb re-walks the hub; the no-upstream case is unpinned by a test
  - id: F2
    severity: LOW
    class: consistency
    location: internal/fabricengine/stencilcommit.go:58
    summary: CommitSeededStencils doc still describes the per-process seeding pass as its caller ("an ordinary run", "pushing per run"), but only lyx stencil sync calls it now
---

### [LOW] reconcileHubWide's unconditional push leaves the stamp absent for a board that cannot push

**Location:** `internal/hubreconcile/walk.go:209`
**Issue:** Focus item 2 asked what happens for a board with no upstream.
`Bolt.pullFirst` returns no skip for it (`ErrNoUpstream` after a successful or a failed fetch), so `PullThenCommitWritten` writes and commits.
`reconcileHubWide` then calls `bolt.Push`, which reaches `PushCoalesced`; `HasUnpushed` treats no upstream as unpushed, and `git push` with `push.autoSetupRemote=true` succeeds against a reachable `origin` and sets the upstream.
The stamp can therefore still be written for that case.
It cannot be written when the board has no reachable push destination: an unreachable `origin` with no upstream, or no `origin` remote at all (which `fabricengine`'s cleanup and weft code treat as a supported shape).
The push fails on every run, even when nothing was committed, because the push is no longer gated on `committed`; `reconcileHubWide` reports `skipped`, the stamp stays absent, and every start verb repeats the whole hub walk with the Warn.
Before the diff, a push failure was non-fatal and the stamp was written after one commit.
Card 10 specifies "a push failure after a landed commit … also reports `skipped`", so the failure-after-commit case is as planned; the extension to a push with nothing committed, and to a board that can never push, is the diff's own and is recorded in `doc.go`.
No test in `ensure_integration_test.go` pins a no-upstream board (reachable or not), although `boltpullfirst_integration_test.go` pins the write side of that case in `fabricengine`.
**Fix:** Decide the intended behavior for a board with no `origin` or no upstream and a failed push, and pin it with a row in the existing `TestEnsure_UnreachableBoardRemoteSkipsHubWideConfigAndLeavesStampAbsent` or its sibling.
If a board that has never had an upstream should still be stamped, gate the stamp-absent rule on the board having an upstream; otherwise state in `doc.go` that such a board is re-walked on every start verb.

### [LOW] CommitSeededStencils comments describe a caller it no longer has

**Location:** `internal/fabricengine/stencilcommit.go:58`
**Issue:** Focus item 4 asked whether the file header and the `BoardWriteLockFile`, `StencilsSubtreeRel` and `SpecsSubtreeRel` comments describe the seeding commit path as it now stands.
The file header (`stencilcommit.go:1`), both `*SubtreeRel` comments and the `BoardWriteLockFile` comment (`junctionnames.go:85`) are accurate: `grep` finds `CommitSeededStencils` called only from `internal/stencilcli/cli.go`, and the lock is also taken by `Bolt.CommitWritten`, `PullThenCommitWritten` and the seed-commit drop, which the group wording covers.
The `CommitSeededStencils` doc comment itself was not updated with them.
It still says an empty path list is "the common case on an ordinary run, and what keeps the seeding pass free", and that it never pushes because "pushing per run would fire a push on nearly every lyx invocation".
The once-per-process pass now commits through `PullThenCommitWritten` in `cmd/lyx/stencilseed.go`, so an ordinary run never reaches this function, and its only caller is the operator-requested `lyx stencil sync`.
**Fix:** Reword the empty-list sentence and the never-pushes rationale for `lyx stencil sync` as the sole caller, for example "a sync that wrote nothing commits nothing and takes no lock", and drop the per-run push reasoning.

## Verdicts on the focus directive

- **PATTERN.md size.**
  `wc -c PATTERN.md` is 16382 bytes, under the 16384 `MaxOverviewBytes`; `lyx gate test ./internal/pattern` passes with `TestOverview_RealFilePassesCheck`, and the diff of `PATTERN.md` against the merge base changes only the `PATTERN-hub-store-gc` and `PATTERN-documentation-lifecycle` lines.
  The gc line names the in-flight skip ("skipped while another pair's session is live"); the lifecycle line only shortened its link text and keeps its constraint.
  No other entry lost its constraint.
  Not a defect.
- **`reconcileHubWide` with no upstream.** See F1.
- **Pruned legacy config paths in `fabriccli/fabric.go`.**
  They do not reach `file_written`: `PullThenCommitWritten` stages the deletion of a `MigratedFrom` path but skips `rec.Append(KindFileWritten, …)` when `os.Lstat` reports it gone, and `boltpullfirst_integration_test.go` ("a deleted path is committed but not recorded as written") pins that.
  The handler no longer appends its own `file_written` entries.
  Not a defect.
- **`stencilcommit.go` and `junctionnames.go` comments.** See F2.

Checks run: `lyx gate test --tags integration` over `hubreconcile`, `fabriccli`, `fabricengine`, `stencilstore`, `gitrepo`, `pairteardown`, `stencilcli`, `pattern` and `cmd/lyx` all pass.
Card 10's `Delete` group is satisfied: no reference to `revisionLabel` remains, and the only `shortRevisionLength` is the one in `buildvcs`.
The `internal/hubreconcile` budget row in `cmd/lyx/testdata/test-budget.yaml` equals the package's top-level test count.
The Publish failure's one failing test, `TestOverview_RealFilePassesCheck`, passes at the current size.
