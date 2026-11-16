---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/battencli/startguard.go:125
    summary: No test pins the hub wiring of the FilingMarked seam, so a missing line or a wrong run ID or location passes the whole suite and the filed run is stuck at Run-Shed again
    fix: In internal/battencli/beamland_integration_test.go TestBeamLanding_ReachesWorktreeCreateSkip, add one subtest that writes shedrun.WriteFilingMark(h.Location, "g") for the seeded grouping project g and asserts hubStartGuard(h.Location, "g").refuse("g", "run", runShed, true, true) returns nil. In the same subtest, before writing the mark, assert the same call returns an error carrying "grouping project", so the subtest pins both sides of the real filing-mark read.
    origin: lens:code-tests
---

### [MEDIUM] The filing-mark seam's hub wiring has no test

**Location:** `internal/battencli/startguard.go:125`

**Issue:** `TestStartGuard_Refuse` sets `FilingMarked` only through a fake (`internal/battencli/startguard_test.go:446`).
The wiring `shedrun.FilingMarked(location, slug)` is the production path of the fix: it must read the mark that raddle's `MarkFiled` writes under the slug in the prime (`internal/loomcli/raddlewiring.go:264-270`).
`TestBeamLanding_ReachesWorktreeCreateSkip` drives `hubStartGuard` over a forged hub and seeds the grouping project `g` (`internal/battencli/beamland_integration_test.go:60`), but no subtest writes a filing mark and resumes under the guard.
Dropping the line, or keying the read by another run ID or location, leaves every test green while the bug the board entry describes comes back.

## Holistic review

The diff makes the first of the board entry's two ideas: `refuseProject` lets a run on record with a standing filing mark through to the task checks, which pass any resume past Worktree-Create.
It covers every non-beam grouping project, not only a beam's.
A beam project is already judged by the beam branch, which passes a resume past Worktree-Create.
`ClearMotifRun` admits a done arras run and removes its filing mark with its run directory, so the done run clears as its doc says.
The mark is read in the prime under the slug, the same path `WriteFilingMark` writes and `battenCommitStatusDeps.SetBoardStatus` and `clearDoneRun` read.
The read error goes through `probeRefusal` with the way forward, so PATTERN-refusal-way-forward holds.
The change description's claims match the diff, and the commit carries no `Pattern-Departure:` trailer and needs none.

## Rejected

- Lens code-tests NIT, no row for a nil `FilingMarked` with `Project` set: only a hand-populated test receiver reaches it, and the rubric's trust-the-caller rule exempts internal misuse from tests.
- Lens code-tests side note, the `dirty` and `parts` fields of `FiledRunUnderItsFamilysProjectResumes` are inert for any resume: the row still pins that a filed run passes with them set, and the change description says no more than that, so it is no false claim.
