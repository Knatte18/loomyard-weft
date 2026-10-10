---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:149
    summary: The acceptance criterion says lyx loom resume proceeds for a run with a dead driver, but resume never starts a driver and still refuses there with the no-live-driver message
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:107
    summary: D6 assumes the recorded parent is slug-free, but hubgeom already forms a slug-bearing parent name for a pair created from a task worktree, so the way forward can name a caller the check itself refuses, and the dependency is recorded only in the support log
  - id: F3
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:47
    summary: D1 names the goto whenever the verify exited non-zero, including when the failure record write failed and was only logged, which leaves the fixer round with no record, the state D1 excludes timeouts to avoid
  - id: F4
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:74
    summary: D3's exception for a running run whose driver died mid conflict session does not say what happens to a conflict session strand that outlived its driver and may still be writing into the pair when D2 aborts and redoes the merge-in
---

### [MEDIUM] Resume does not proceed for a dead driver, as the acceptance criterion claims

**Location:** `_lyx/discussion/decision-record.md:149` (Acceptance criteria, fifth bullet), with D3 at line 74.

**Issue:** The criterion says `lyx loom resume` and `lyx loom start` both proceed for a run at Publish or Finalize over a parent-sourced parked merge-in, "both halted there and `running` there with a dead driver".
In `internal/loomcli/resume.go` a run with no live driver goes through `refuseOverUnfinishedMerge` and then always refuses with `noLiveDriverMessage` ("the run has no live driver to wake, and resume starts none"), naming `lyx loom start` or `lyx batten run <slug>`.
So D3's exception, applied to resume, only swaps the `merge_in_progress` refusal for the no-live-driver refusal over a dead driver; resume proceeds only when a parked, live driver exists.
D3's own Rationale gets this right ("a dead driver is respawned only through `start`"), so the record contradicts itself, and a plan or test written from the criterion would either fail or push the implementer into making resume spawn a driver.

**Fix:** Split the criterion: `lyx loom start` proceeds for a run halted or `running` at Publish or Finalize over a parent-sourced parked merge-in with a dead driver; `lyx loom resume` proceeds over it when a parked driver is live, and with no live driver reports the no-live-driver refusal instead of `merge_in_progress`.
State the same split in D3's Decision.

### [MEDIUM] D6's slug check can refuse the run's own recorded parent

**Location:** `_lyx/discussion/decision-record.md:107` (D6 Rationale), the D6 Decision's way-forward sentence, and `_lyx/discussion/support-log.md:77` (Question ledger).

**Issue:** D6 rests on "the orch's strand name is slug-free", and its way forward names "the run's recorded parent" as the caller who runs the verb.
`hubgeom.decideParent` (`internal/hubgeom/parent.go`) already forms the parent as `agentname.Format(shortname, slug, RoleOrch)` with the origin's `ParentWorktree` as slug whenever that worktree is not the prime, and `fabricengine` records the creating worktree as `ParentWorktree` in `add.go`.
A pair created from a task worktree therefore has a recorded parent `<shortname>:<slug>:orch`.
For such a run, the refusal names that parent as the one to run the verb, while the same check refuses any caller whose name carries a slug.
No orch runs outside the prime today, so the named parent cannot answer at all, and once `orch-any-worktree` lands it would be refused.
The support log's ledger says "today it is `<shortname>:orch`" and notes the dependency, but `Plan-Write` never reads the support log, so the dependency and its consequence are missing from the decision record.

**Fix:** Make the check tell the recorded parent apart directly: let a caller whose strand name equals the run's recorded parent name pass, and refuse every other named caller in a task pair.
That keeps the bound and drops the slug-free premise.
If the slug rule stays, the way forward names the operator whenever the recorded parent carries a slug, and the decision record's Open risks carries the `orch-any-worktree` dependency.

### [LOW] D1 names the goto when the failure record was not written

**Location:** `_lyx/discussion/decision-record.md:47` (D1 Decision), with the first acceptance criterion and the `Publish: verify failed` row.

**Issue:** `verifyGate.recordFailure` in `internal/landingshed/verifygate.go` writes a record only for `StatusFailed` with `TimedOut` unset and `ExitCode >= 0`, which matches D1's split: a non-zero exit writes one, a timeout and an unstartable shell write none.
`WritePublishFailure` can still fail, and `recordFailure` then logs a warning and returns, while `verdict` still produces the failed-verify reason.
Under D1 that reason names the `Webster-Burler` goto although the round has no record to read, which is the "judges the diff blind" state D1 gives as its reason to keep the goto off the timeout reason.

**Fix:** State in D1 which way forward a non-zero exit gets when its record write fails: either the goto is named only when the record was written, else the reason stays as today, or the Bound accepts a blind round in that case.

### [LOW] D3 is silent on a conflict session that outlived its driver

**Location:** `_lyx/discussion/decision-record.md:74` (D3 Decision).

**Issue:** D3 lets `lyx loom start` through for a run still `running` at Publish or Finalize "whose driver died mid conflict session".
The conflict session is its own shuttle strand (`mergeresolve` spawns it through `Shuttle.Run`), so a driver's death does not end it.
After `start`, the re-stepped row aborts the parked merge-in (D2) and runs the clean check and a fresh merge-in, while the old session may still be editing the conflicted paths.
Its writes can then trip the clean check into the "uncommitted changes" halt #471 removes, or land in the new merge's files.
Before D3 the refusal forced the same hand abort, so this is not a new race, but D3 is the Decision that makes the path routine and it neither bounds it nor names a way forward for it.

**Fix:** State in D3 whether the row ends a live conflict session of its own run before the abort.
If it does not, say so in the Bound, along with the way forward when the clean check then trips.

## Focus verdicts

- D1 against `recordFailure`: a record is written for a non-zero exit only, and none for a timeout or an unstartable shell.
  The Decision, the Goal's first bullet, the first acceptance criterion and the spec row agree with that, apart from the write-failure gap in F3.
- D3 against `refuseOverUnfinishedMerge`: the guard takes no producer or parent branch today, and the plan note covers that.
  For a `running` run with a dead driver, `start` reaches the guard on its spawn path and proceeds under D3.
  Resume refuses anyway (F1).
  The way forward for the dead-driver case is the existing `noLiveDriverMessage`, which names `start`, so it needs no new text apart from F4.
- D6 in a standalone run: agents are named `s<hash>:<role>` with an empty `NameSlug` and no parent.
  D6 refuses any non-empty name there and names the operator, and the plan stencil text covers the no-parent case with "stop and report", so both are correct there.
- D6 session name: the Decision and the acceptance criterion forbid a literal shortname and name the recorded parent, which is resolved per hub through `hubgeom.ResolveParent`, so no stencil spells one.
  The defect is which parent name comes back (F2), not literal spelling.
- D2's abort against a hand-started `lyx fabric merge-in` of the parent branch: the Bound names that case and its loss of any partial resolution, and limits the abort to that verb and source.
  The resolver's Stuck paths abort before halting (`abortAndStuck`), so no way forward tells an operator to resolve a parked merge-in by hand.
  The stated bound is sufficient, and this is not a defect.
