# Card 10 — darn-write-entry-reroute

**Create:**
- `internal/loomshed#darnWrite.rerouteReason`
- `internal/loomcli#mergeableTips`
- `internal/loomcli#parentTipsReader`
- `internal/loomcli#TestMergeableTips`

**Edit:**
- `internal/loomshed#DarnDeps`
- `internal/loomshed#darnWrite.Call`
- `internal/loomshed#darnWrite.checkSeams`
- `internal/loomshed#darnResumeWayForward`
- `internal/loomshed#newDarnFixture`
- `internal/loomshed#TestDarnWrite_Call`
- `internal/loomshed/doc.go`
- `internal/shedrecipe#darnWriteEntry`
- `internal/shedrecipe#darnTestEnv`
- `internal/shedrecipe#TestDarnWriteEntry_ConstructionFailures`
- `internal/testkit/envkit#FullEnv`
- `internal/loomcli#darnDeps`
- `internal/loomcli#landingDeps`
- `internal/loomcli#intentionallyNil`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestDarnDeps_PublishFailureNamesThePresentRecord`
- `internal/battencli#intentionallyNil`

**Uses:**
- `internal/loomcli#seedRunStart`
- `internal/loomcli#readRecordedParentBranch`
- `internal/verifytree#DirtyPaths`

**Intent:** D3: a Darn-Write call that finds this run's failure record, a task branch that has not moved since it, a clean task worktree and a moved parent returns a reroute to Merge-In without spawning the writer, so an owned failure the parent fixed since is merged in instead of looping through the writer's gate.

`DarnDeps` changes, each field documented like the others:
- `PublishFailure` becomes `func() (path string, record verifytree.PublishFailure, found bool, err error)`, so the producer reads the record's `written_at` and `head` beside its path;
- new `RunStart func() (time.Time, error)`, the run's start;
- new `TaskHead func() (string, error)`, the task branch's HEAD;
- new `Dirty func() ([]string, error)`, the task worktree's uncommitted paths, read through the dirty-paths reader Merge-In's clean check uses;
- new `ParentTips func() ([]string, error)`, the tips Merge-In would merge, read with no fetch;
- new `IsAncestor func(sha string) (bool, error)`, whether sha is an ancestor of HEAD.
`checkSeams` names each new seam when it is nil, and `darnWriteEntry`'s seam list checks them, so a darn wiring that leaves one unset is refused at build, naming it.
The seams are told, never read here: `internal/loomshed` may not import `internal/fabricengine`.

`rerouteReason` returns "" for no reroute, checking D3's four facts in order:
the record's `WrittenAt` parses and is at or after `RunStart`, compared as whole-second UTC, a zero `RunStart`, which `seedRunStart` returns for a seed with no stamp, counting as not this run's, since the record's run cannot be told;
`TaskHead` equals the record's `Head`;
`Dirty` returns no path;
and some tip `ParentTips` returns is not an ancestor of HEAD.
The seam always returns the local tip, so short of every tip being an ancestor, a failed tip or ancestor read is the only not-moved case.
A `RunStart`, `TaskHead`, `ParentTips` or `IsAncestor` error is logged at Warn and counts as no reroute; a `Dirty` error is logged at Warn and counts as dirty; in every such case the writer runs as today.
A reroute's reason says the parent branch moved since the verify failure the record at `recordPath` names, while the task branch and worktree have not changed since it, and that the run reroutes to Merge-In, which merges the parent and re-runs the failing gate.
It adds that if Merge-In stops without merging, on a parent pair that is dirty, mid-merge, ahead of or diverged from its upstream, the way forward is to repair the parent pair as Merge-In's Stuck reason says before any resume, since a resume over the unrepaired pair reroutes back to the same stop; a reroute-cap halt embeds this reason, so the parent reads it before the cap halt's resume.
It carries no `lyx loom resume` way forward of its own.

`darnWrite.Call` reads the record through `PublishFailure` as before; when a record is present, it calls `rerouteReason`, and a non-empty reason returns `Stuck` with `OutputPointer{Reason: reason, Reroute: true}` before the session is built, so no writer is spawned, nothing is committed and no bounce budget is spent (shed's reroute arm); card 9's edge takes it to Merge-In.
The renderPriorWork call takes the path from the new seam shape.
`darnResumeWayForward` becomes: `way forward: run "lyx loom resume" in the task worktree, which re-spawns Darn-Write with a fresh verify budget, or, when the parent has moved since this run's verify failure and the task branch and worktree have not, first takes the parent in through Merge-In`.
`doc.go`'s Darn-Write paragraph states the four conditions, the reroute's reason and the reroute cap that halts a row rerouted back from a Merge-In stuck on a parent pair that is dirty, mid-merge, ahead of or diverged from its upstream.

Bound: the reroute skips the Darn-Review perch and Describe only while the task branch carries no commit and the worktree no uncommitted edit since the failed verify, which ran on reviewed code, so the only new code is the parent merge; a writer that committed or left edits disables it, a passing Merge-In or Publish removes the record, and after Merge-In merges the tip is an ancestor of HEAD, so the next entry does not reroute.

`parentTipsReader` is the one parent-tip reader both landing and darn read through, so D3's check shares landingshed's tip-reading seam: it opens the fabric at call time and calls `ReadParentTips(parentBranch, fetch)`.
`landingDeps` sets `Deps.ReadParentTips` to `parentTipsReader(l, parentBranch)` in place of its inline closure.
`loomcli`'s `darnDeps` fills the seams over location: `PublishFailure` returns the record with its path; `RunStart` is `seedRunStart(l)`; `TaskHead` opens the fabric and reads `HeadSHA`; `Dirty` is `verifytree.DirtyPaths(l.WorktreePath())`; `ParentTips` reads the parent through `readRecordedParentBranch(l)`, the origin record `landingBranches` resolves the landing parent from, calls `parentTipsReader(l, parent)(false)` and returns `mergeableTips(tips, fabricengine.EnvSyncOptions().SkipPush)`; `IsAncestor` opens the fabric and calls `IsAncestor(sha, "HEAD")`.
`mergeableTips` returns the local tip alone when pushes are skipped, and the local tip and the non-empty upstream tip otherwise, empty tips dropped.

The five new seams are nil on a loom seed and in batten's Env, so `internal/loomcli#intentionallyNil` gains `Darn.RunStart`, `Darn.TaskHead`, `Darn.Dirty`, `Darn.ParentTips` and `Darn.IsAncestor` as "darn-only seam, filled by wire for a darn seed", and `internal/battencli#intentionallyNil` gains them as "darn-only seam, batten has no darn segment", keeping both `TestWire_EverySeamFilled` tests green.

Tests:
- `newDarnFixture` sets the new seams to an unmoved, clean, current state: `RunStart` before the record, `TaskHead` equal to the record's head, no dirty path, one parent tip that is an ancestor; `PublishFailure` returns no record by default, so the existing rows spawn the writer as today.
  The fixture gains a `rerouteReady` mutation that presents this run's record, HEAD at its head, a clean tree and one parent tip outside HEAD, the state that reroutes.
- `TestDarnWrite_Call` gains the reroute row: `rerouteReady` returns `Stuck` with `Reroute` set, a reason naming Merge-In and the parent-pair repair and not containing `lyx loom resume`, no events and no session built.
  Every writer-runs row starts from `rerouteReady` and flips one fact, so it fails if that fact were ignored: no record, a record written before the run's start, a record whose `WrittenAt` is empty, a zero `RunStart`, a HEAD past the record's head, a dirty tree, a failed status read, a failed run-start read, a failed HEAD read, a parent tip already in HEAD, a failed tip read and a failed ancestor read; each asserts `session` true and events `session,commit`.
  A row with two parent tips, the first in HEAD and the second outside it, reroutes, so the scan reads past the first tip.
  The gate-failed row asserts its reason holds the literal words `lyx loom resume` and `Merge-In` after the session's own reason, not only the constant, so the new text's Merge-In clause is pinned.
  The prior-work row reads the new seam shape; unwired-seam rows name `RunStart`, `TaskHead`, `Dirty`, `ParentTips` and `IsAncestor`.
- `TestMergeableTips` covers a skipped push with an upstream tip outside HEAD, which returns the local tip alone, so the writer runs as today on such a run; an unskipped push with both tips; and an empty upstream.
  It tests the helper directly because `darnDeps`' `ParentTips` opens a real fabric and `TestDarnDeps_PublishFailureNamesThePresentRecord` runs over a bare temp location with none, so no public-surface test of the closure can reach the push-skip choice without a hubforge pair.
- `TestDarnDeps_PublishFailureNamesThePresentRecord` asserts the new shape: no record reads not found, a written record reads back with its path.
- `TestWire_DarnRecipe`'s `darnSeams` list gains the five new seams, so a darn wiring that leaves one nil fails it.
- `darnTestEnv` and `FullEnv` fill the new seams, and `TestDarnWriteEntry_ConstructionFailures` gains a row per new seam set to nil, naming `Darn.<seam>`.
Card 9's `latestDarnOutcome` change keeps the next writer spawn's prior-work note on the last non-reroute outcome.

**ImpactSummary:** `DarnDeps` gains five seams and re-types one, which reaches the loomshed fixture, `darnWriteEntry`'s seam list, two Env fixtures, loomcli's `darnDeps` and both seam-guard lists, all on this card; `landingDeps` moves its tip closure onto the shared reader with no behaviour change, and the producer's existing paths keep their behaviour when the reroute does not apply.
