# Card 12 — A green check lifts its subject's holds and starts the runs

**Create:**
- `plan:internal/adoptcli#greenDeps` -> `type greenDeps struct`
- `plan:internal/adoptcli#greenEnd` -> `func greenEnd(deps greenDeps) []string`
- `plan:internal/adoptcli#hubGreenDeps` -> `func hubGreenDeps(prime *lymcwd.Location, subject string) (greenDeps, error)`
- `plan:internal/adoptcli#TestGreenEnd` -> `func TestGreenEnd(t *testing.T)`

**Edit:**
- `internal/adoptcli#checkDeps`
- `internal/adoptcli#runCheck`
- `internal/adoptcli#hubCheckDeps`
- `internal/adoptcli#newCheckFixture`
- `internal/adoptcli#checkFixture`
- `internal/adoptcli#TestRunCheck`
- `internal/adoptcli/doc.go`
- `internal/landingshed#ownerHaltReason`
- `internal/battenshed/doc.go`
- `docs/overview.md`

**Extends:**
- `PATTERN.md`

**Uses:**
- `plan:internal/adoptcli#readHolds`
- `plan:internal/adoptcli#removeHolds`
- `plan:internal/adoptcli#writeTries`
- `internal/adoptcli#subjectDir`
- `internal/pauserecord#HoldCaller`
- `internal/fronttask#FrontOf`
- `internal/fronttask#Remove`
- `internal/fronttask#HubDeps`
- `internal/fronttask#RemoveError.HandRemoval`
- `internal/fronttask#ActionAdoptCheck`
- `internal/adoptcli#resumeChild`
- `internal/adoptcli#readStatus`
- `internal/landingshed/doc.go`

**Intent:**
D5's green end, which takes over the resumes card 11 took out of adopt.

`checkDeps` gains `Green func() []string`, nil doing nothing; `runCheck` calls it once the gate exits 0, after the green record is written, and posts each line it returns to the orch as one notice after the green notice.
A failed end, an end before any tier, and a check whose owner reads elsewhere call it never.

`greenDeps` holds the subject, its directory, its worktree name (the prime's for `root`) and project (none for `root`), and the seams `PauseRecord`, `Lift` (`tenderengine.Lift` under the `adopt` caller label), `BeamUp` (`orchcli.BeamUp` of the subject's worktree, PATTERN-dobby-orch-pair), `RunRecord`, `StartFirstPass` (`battencli.StartFirstPass` from the prime), `Tasks`, `ParentOf`, `ChildStatus`, `ResumeChild` (`resumeChild`, its spawn logged), `Owner` (the effective owner of a slug, against this machine) and `FrontDeps` (`fronttask.HubDeps(prime)`); `hubGreenDeps` fills them over the hub and `hubCheckDeps` sets `Green` to `greenEnd(hubGreenDeps(prime, subject))`, a build error being the one line it returns.

`greenEnd`, in order, each failure a returned line naming its way forward while the other steps go on:
1. reads the subject's holds file; for each listed slug whose own scope still holds a live `HoldCaller` entry, lifts it by scope, caller and pause time; then `removeHolds` drops every slug it read there in one update, so a slug a second adopt appended meanwhile stays listed and is lifted by its own check (bound: that adopt's reconcile may have moved the tree under this check).
   A failed lift names `lym tender resume <slug>`.
2. brings the subject's dobby and orch up through `BeamUp`, which leaves a live pair alone; a failure names `lym beam start <subject>`.
3. for each lifted slug with a batten run record that is not done, starts its detached first pass; a start racing the dobby's own pass at the lift reads ok, since the driver-lock refusal happens inside the detached pass, which loses the lock and exits, so the race needs no branch; a refusal envelope (covered, run status or owner) names `lym batten run <slug>` from the prime.
   A lifted released task with no record is left to the dobby, which starts it now the pause is gone.
4. for each task entry not done whose parent chain reaches the subject, the prime for `root`, without passing another beam, whose child status is blocked with an error opening `landingshed.OwnerHaltPrefix` and whose effective owner is this machine, runs `lym loom resume` in its pair; a failure names `lym loom resume` in that pair.
5. removes the subject's fix task, `fronttask.FrontOf(tasks, project, ActionAdoptCheck)`, when one stands not done in any status, draft, released, running or halted, through `fronttask.Remove`, whose cancel step ends a running one; a failure carries the step's error and its `HandRemoval`.
6. writes the subject's try count to 0.

Bound: a hold lifted by hand (`lym tender resume`, or `lym batten cancel <slug>` through `liftSlugPauses`) is gone at the green end, which then lifts nothing for it; `--keep-local` over a live pair left that run unheld.

Tests:
- `TestGreenEnd` over fake seams and a temp adopt directory holding two subject directories: two listed holds standing are lifted, the holds file is left without them, and one appended by a concurrent adopt after the read stays; the other subject's listed hold and its `HoldCaller` entry stay; a hold lifted by hand meanwhile is skipped; a failed lift returns a line naming `lym tender resume <slug>` and the other steps go on; the pair is brought up, and a failed `BeamUp` returns a line naming `lym beam start <subject>`; a first pass starts per lifted run with a record and none for a released task without one; a refusal envelope returns a line naming `lym batten run`; a child halted at Finalize's owner check under the subject is resumed and one halted for another reason, one under another beam and one whose effective owner is another machine are not; a fix task standing draft, released, running or halted is removed, a released one that never started included; a failed removal's line names its hand removal; the try count reads 0 after.
- `TestRunCheck` gains rows: a green end calls `Green` once and posts its lines after the green notice; a failed end and an end before any tier never call it; a check that ends green and one that ends failed each leave a `holds.json` and a `fix-tries.json` written in the subject directory beforehand byte for byte as they were (D4, D6).
  A `hubGreenDeps` build error is the one line `Green` returns, a branch `hubCheckDeps` holds over the hub, which `TestHubCheckDeps_StaleStampRunsTheTunedFinalizeTiers`'s hub fixture does not reach; it is a propagated raw read error, exempt as transient.

`PATTERN.md` gains, in `## Shed and loom` after `PATTERN-front-task`:

```
- `PATTERN-adopt-hold` — On a takeover, `lym adopt` holds it, and each unstarted released task of the subject bar a front task, with a `pauserecord` hold-caller entry; only the green adopt check of the subject lifts it, never dobby; by hand, `lym tender resume` or `lym batten cancel`.
```

`adoptcli/doc.go`'s check section states the green end's steps and bounds.
`ownerHaltReason`'s refusal for a parent owned elsewhere says the adopt's green check resumes the halted run, in place of the adopt resuming it.
`battenshed/doc.go`'s line on detached first passes names the adopt check's green end, not `lym adopt`, beside `lym beam land`, as starting a first pass detached.
`docs/overview.md`'s batten paragraph that says `lym adopt` starts a run's first pass detached names the adopt check's green end in its place, beside `lym beam land`.

**ImpactSummary:** runCheck gains one seam called at green; the new green end lifts, starts, resumes and removes inside adoptcli over existing tender, batten, orch and fronttask calls; landingshed's owner refusal, battenshed's doc and the overview's batten paragraph name the green end as what resumes and starts.

**Commit:** 12: adopt check: a green end lifts the holds and starts the runs
