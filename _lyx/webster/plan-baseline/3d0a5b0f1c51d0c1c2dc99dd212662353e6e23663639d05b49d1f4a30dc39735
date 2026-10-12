# Card 20 — guarded-landing-rows

**Create:**
- `internal/landingshed/guardedrows_test.go`

**Edit:**
- `internal/landingshed#Publish`
- `internal/landingshed#NewPublish`
- `internal/landingshed#Publish.Call`
- `internal/landingshed#PRGate`
- `internal/landingshed#NewPRGate`
- `internal/landingshed#PRGate.Call`
- `internal/landingshed#Finalize`
- `internal/landingshed#NewFinalize`
- `internal/landingshed#Finalize.Call`
- `internal/landingshed#Finalize.pushParent`
- `internal/landingshed#newTestDeps`
- `internal/landingshed#newPRGateFixture`
- `internal/landingshed#TestNewPRGate_Refusals`
- `internal/landingshed#TestNewFinalize_Refusals`
- `internal/landingshed#TestFinalize_MergeInRequiredRetry`
- `internal/landingshed/doc.go`
- `PATTERN.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Delete:**
- `internal/landingshed#Publish.noteMergeCommit`

**Uses:**
- `internal/landingshed#newParentGuard`
- `internal/landingshed#parentGuard.ensure`
- `internal/landingshed#parentGuard.release`
- `internal/landingshed#parentGuard.moved`

**Intent:**
Make Publish, PR-Gate and Finalize hold the parent lock and reroute to Merge-In on a moved parent (D5, D9), and add the invariant they share (D14).

Each of the three gains a `guard parentGuard` built by its constructor.
Each constructor refuses a nil `Deps.Lock.Store`, a nil `Lock.Owner` and a nil `ReadParentTips` with its own message.
A zero guard, as a struct-literal test builds, takes no lock and reads no tips, as the zero pre-merge step runs no probe.

- Publish, for a parent in `require_pr_to_base`, ensures the lock and then calls `moved` before `publish_verify` and the push.
  A moved parent returns Stuck with `Reroute` set and the reroute reason.
  A parent outside the list stays a no-op, and so does PR-Gate's first row: neither touches git or GitHub, and Finalize's check covers the landing.
- PR-Gate, for a parent in the list, ensures the lock and checks `moved` before it queries the pull request, so the orch never decides on a pull request whose base moved.
  A re-call after a resume takes a record handed to batten back, since an acquire by the same run is granted.
  Its matching-rejection Stuck releases the run's lock before it returns, the edge out of the Merge-In-to-Finalize window to PR-Rework (darn: Darn-Write), and the reworked run reacquires at Merge-In.
- Finalize ensures the lock and checks `moved` after parsing the change description and committing the status file, before its catch-up merge-in.
  A parent that already holds the task's work, after a GitHub merge or an earlier landing, reads as not moved and lands through the already-landed path with no reroute.
  The catch-up merge-in stays, and since the local tip is checked equal to the merged tip under the lock, it brings no parent code the merge review has not covered while still syncing the records side; the parent-records commit keeps its place after it.
  On the parent-side merge's merge-in-required error, Finalize calls `moved` again, the same two-ref read.
  The single retry runs only when the parent has not moved, which covers a parent moved by run records alone; otherwise Finalize reroutes, so a code-side commit that arrives between the check and the parent-side merge, local or pushed, never lands.
  `pushParent` releases the run's lock on its Done, before Friction-Reflect runs.
- The lock's paused and unreadable reasons return as Stuck with `Halt` set at all three rows.

`PATTERN.md` gains, under `## Fabric and git`:

```
- `PATTERN-parent-lock` — Merging a parent into a task branch, or landing on it, from a run: only under that parent's lock through `internal/parentlock`; a moved parent reroutes to Merge-In instead of merging new code at a later row.
```

`landingshed/doc.go` documents the lock and the reroute at each row.
`Publish.noteMergeCommit`, unused since card 18, is deleted.
`newTestDeps` and `newPRGateFixture` fill a temporary lock store and tip reads equal to a written merge record, so the existing producer tests run unchanged in substance, and the two refusal tests gain the new rows.
`TestFinalize_MergeInRequiredRetry` gains the moved-again case.

`guardedrows_test.go` drives the three rows with fake tips and a real temporary store:
- Publish reroutes on a moved upstream;
- PR-Gate reroutes before any GitHub call, and its rejection releases the lock;
- Finalize reroutes on a moved parent, lands with no reroute when the parent holds the work, and retries or reroutes after merge-in-required as the second read decides;
- Finalize's Done leaves no record;
- a moved parent at each row returns a reroute Stuck, which spends no bounce budget;
- a pause during a lock wait halts the row blocked with its way forward.

**ImpactSummary:** the three landing rows now take the lock and reroute on a moved parent; their constructors gain three refusals, and the shared fixtures fill the lock seams.

**Commit:** 20: landing: Publish, PR-Gate and Finalize hold the parent lock and reroute on a moved parent
