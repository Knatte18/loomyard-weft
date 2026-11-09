# Card 4 — green-end-reports

**Delete:**
- `internal/adoptcli#liftHolds`
- `internal/adoptcli#startLiftedRuns`
- `internal/adoptcli#resumeHaltedChildren`
- `internal/adoptcli#entryDone`
- `internal/adoptcli#resumeChild`
- `internal/adoptcli#greenFixture.hold`
- `internal/adoptcli#greenFixture.holds`

**Edit:**
- `internal/adoptcli#greenDeps`
- `internal/adoptcli#greenEnd`
- `internal/adoptcli#hubGreenDeps`
- `internal/adoptcli#hubCheckDeps`
- `internal/adoptcli#fixReadError`
- `internal/adoptcli#fixFilingError`
- `internal/battencli#OwnedElsewhereError.Error`
- `internal/adoptcli#greenFixture`
- `internal/adoptcli#newGreenFixture`
- `internal/adoptcli#TestGreenEnd`
- `internal/landingshed#ownerHaltReason`
- `internal/beamcli#createCmd`
- `internal/beamcli/doc.go`
- `internal/battenshed/doc.go`
- `docs/overview.md`

**Uses:**
- `internal/landingshed/doc.go`

**Intent:**
D8: a green adopt check only reports.
`greenEnd` keeps two steps: it removes the subject's fix task (`removeFixTask`) and writes the subject's fix-try count to 0, each failure a returned line naming its way forward as today.
It no longer lifts pauses, brings up the dobby and orch, starts first passes or runs `lym loom resume` on owner-halted children.
The red end and `runCheck` are unchanged.

`greenDeps` drops the seams only those steps read: `PauseRecord`, `Lift`, `BeamUp`, `RunRecord`, `StartFirstPass`, `ParentOf`, `ChildStatus`, `ResumeChild` and `Owner`, and `hubGreenDeps` stops filling them.
`liftHolds`, `startLiftedRuns`, `resumeHaltedChildren`, `entryDone` and `resumeChild` lose their only callers and are deleted.
`green.go`'s file comment and `greenEnd`'s doc say the green end removes the fix task and resets the try count.
`hubCheckDeps`'s fix-loop build-error line drops "the holds standing meanwhile" for "the adopt pauses standing meanwhile".
`fixReadError`'s and `fixFilingError`'s texts end the same way, with the adopt pauses standing meanwhile in place of the holds.

Every text that promises the green end's starts or resumes is reworded with it:
- `ownerHaltReason`'s `runOwner == ""` way forward drops "whose green adopt check resumes the halted run".
  It says the adopt here pauses the run, and that `lym tender resume <slug>` and then `lym loom resume` in its pair let it go on.
  A grep of the repository for "resumes the halted run" finds no test asserting the old text; the landingshed `doc.go` does not quote it.
- `createCmd`'s `Long` text and the beamcli `doc.go` drop "the green end of the adopt check that `lym adopt` starts" from who starts or resumes a beam's run.
- The battenshed `doc.go` drops the green end of the adopt check from who starts a run's first pass detached.
- `OwnedElsewhereError.Error`'s text drops the green adopt check that starts the run.
  It says `lym adopt <subject>` pauses its run, which `lym tender resume <slug>` lets go on.
  No test asserts the old text.
- `docs/overview.md` drops "the subject's green adopt check lifts the holds" and "The adopt check's green end starts a run's first pass detached".
  Card 10 rewrites its adopt entry whole once the worker, the record and the step order exist.

`TestGreenEnd` (tier0) drops the subtests of the lift, the bring-up, the first passes and the child resumes, and the fixture drops their fields and the `hold`/`holds` helpers.
It keeps the fix-task removal and try-count subtests, which carry D8's kept half.
D8's other half holds by construction: `greenDeps` keeps no seam that lifts, starts or resumes, so no row could observe such an effect.

**ImpactSummary:** the green end and its deps shrink inside adoptcli, the fix-loop texts name the adopt pauses, and a landingshed and a battencli refusal text and the docs drop the green end's starts; `runCheck`, the fix loop's logic and every other caller are untouched.
