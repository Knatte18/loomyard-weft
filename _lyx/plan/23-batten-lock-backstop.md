# Card 23 — batten-lock-backstop

**Create:**
- `plan:internal/battenshed#ChildParentLock` -> `type ChildParentLock struct`
- `plan:internal/battenshed#innerRunProducer.lockStep` -> `func (p *innerRunProducer) lockStep(ctx context.Context, w *childWait)`
- `plan:internal/battenshed#innerRunProducer.staleBaseResume` -> `func (p *innerRunProducer) staleBaseResume(ctx context.Context, w *childWait) *waitEnd`
- `internal/battenshed/innerrun_parentlock_test.go`
- `plan:internal/battencli#childParentLock` -> `func childParentLock(slug string, locateTask func() (*lyxcwd.Location, error), boardDir string, bound time.Duration) *battenshed.ChildParentLock`
- `plan:internal/battencli#TestChildParentLock` -> `func TestChildParentLock(t *testing.T)`

**Edit:**
- `internal/battenshed#InnerRunDeps`
- `internal/battenshed#innerRunProducer`
- `internal/battenshed#innerRunProducer.check`
- `internal/battenshed#innerRunProducer.awaitingStep`
- `internal/battenshed#battenshedAllowedImports`
- `internal/battenshed/doc.go`
- `internal/battencli#battenCLI.wire`
- `internal/testkit/envkit#FullEnv`
- `internal/loomcli#intentionallyNil`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/parentlock#IdleWatch.Observe`
- `plan:internal/parentlock#Store.List`
- `plan:internal/parentlock#Store.Release`
- `plan:internal/shuttleengine#ReadAgentWaits`
- `plan:internal/fabricengine#Fabric.ReadParentTips`
- `plan:internal/fabricengine#Fabric.HoldsWork`
- `plan:internal/landingshed#ReadMergeRecord`

**Intent:**
Let batten keep a lock handed to it at PR-Gate, backstop a loop that failed to release (D8), and resume a child whose base moved while it awaits the orch (D10).

`InnerRunDeps` gains `ParentLock *ChildParentLock`; nil switches all of this off.
`ChildParentLock` carries six told seams:
- `Read`, the record naming the child's run;
- `Release(rec)`, a compare-and-release on `rec`'s owner;
- `Agents`, the child pair's agent states;
- `SelfPID`, this batten's pid;
- `IdleBound`, landing's `parent_lock_idle_release_min`;
- `ParentMoved`, whether the child's parent upstream moved.
`battenshedAllowedImports` admits `internal/parentlock`.

`lockStep` runs in `check` on every probe check, at notice-probe cadence, and acts only on a record naming the child's run.
Every release logs one Warn.
- A record handed to batten (its owner pid is `SelfPID`) is kept while the child awaits at PR-Gate and is released once that awaiting has lasted `IdleBound` on batten's `Awake` clock with no decision record.
  It is kept while the child runs after a resume, so the resumed loop's PR-Gate takes it back.
  It is released when the child is in any other state: the child left awaiting for a stop other than a resume, or a resumed child stopped again without taking it back.
- A record owned by the child's loop is released when that owner, by pid and start time, has met the idle rule.
  The rule is an `IdleWatch` fed `Agents` on the `Awake` clock and reset whenever the owner changes, so the release never frees a record a fresh loop took after a resume.
  A failed agent read releases nothing.
- Bound: batten releases only records naming its own child's run, and only under the bound or a matched stalled owner.

`staleBaseResume` runs in `awaitingStep` for a child awaiting at PR-Gate with no fresh decision, at notice-probe cadence.
When `ParentMoved` reports true it resumes the child through `Spawn` as it resumes after a decision, so PR-Gate's re-call reroutes to Merge-In under the lock (D9).
It suppresses the awaiting notice for that episode.
A refused resume is retried as a decided one is; an error from `ParentMoved` is warned about and resumes nothing.
Bound: batten only resumes, the reroute decision stays PR-Gate's, and the fetch runs only for a child awaiting at PR-Gate.

In battencli, `childParentLock` builds the seams:
- `Read` lists the store at `parentlock.Dir(boardDir)` for the record whose owner slug and pair are the child's;
- `Agents` maps `ReadAgentWaits` over the task pair;
- `SelfPID` is `os.Getpid()`;
- `ParentMoved` reads the task pair's merge record under `loomengine.LoomScratchDir`, then `ReadParentTips(<the record's parent_branch>, true)`, where a parent with no upstream, or no record, reports false; it compares the upstream tip with the record's `merged_tip`, and a tip that `HoldsWork` the task head reads as not moved.
`wire` fills `InnerRun.ParentLock` with the landing config's `IdleRelease()`.
`envkit.FullEnv` fills it, and loomcli's `intentionallyNil` names it as a batten-only seam.

`innerrun_parentlock_test.go` drives the wait with a fake clock and seams:
- a handed record is kept within the bound and released past it with no decision;
- it is kept after a decision resumes the child, and released when the resumed child halts while still handed;
- a loop-owned record is released after the idle rule holds, and kept when the owner changed, beside a busy agent or on a failed read;
- a moved parent resumes an awaiting child once and sends no notice.
`TestChildParentLock` asserts the record lookup by run over a temporary store and the moved-parent read over fake tips.

**ImpactSummary:** batten's InnerRun wait gains a probe-cadence lock step and a stale-base resume, both off while `ParentLock` is nil; batten's wiring and the seam lists follow.

**Commit:** 23: batten: hold a PR-Gate lock, backstop a stalled loop, resume on a moved base
