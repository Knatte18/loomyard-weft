---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lym/landing/summary.md#Merge danger
    summary: The Merge danger section calls the change a two-way door with blast radius dobby, yet the diff changes what beam.land_notice off means for a beam member under a recorded pull request and drops the member key from the dobby state record.
    fix: "In _lym/landing/summary.md under ## Merge danger, change **Door:** to one-way and name the change that sets it: beam.land_notice off now suppresses a beam member's landing notice under a recorded pull request, which it did not before, and LandNoticeState no longer writes the member key. Widen **Blast radius:** from dobby to dobby and the orch's beam pull request upkeep, since an orch no longer gets the prompt to run lym beam describe for a member whose land_notice is off (contracts/orchrole/role-beams.md:55-56)."
    origin: assist:assist
---

### [MEDIUM] Merge danger calls a setting-meaning change two-way

**Location:** _lym/landing/summary.md#Merge danger
**Issue:** The rubric makes a `two-way` call a finding when the diff changes what a config setting means or changes a record format another binary reads.
Before this diff, `beam.land_notice` off still sent a beam member its landed line under a recorded pull request (the removed condition at `internal/dobbyengine/landnotice.go:118` of the base); now it sends nothing.
The diff also drops `LandNoticeState.Member`, so the dobby state record no longer carries `member`.
A revert stays safe, since an older binary re-reads membership from the board each tick, but the summary names neither change, and its blast radius of `dobby` leaves out the orch's pull request description upkeep that loses its prompt.

## Holistic review

Section 1 of the Goal lands in full:
`rememberRuns` decides a line from `note.Requested` alone, the follow-up stays on that line, `Member`, `isBeamMember`, `landNoticeTasks` and `landNoticeBoard` are gone with no remaining reader,
and the package doc, the `landnotice.go` header and the doc comments of `landNotices`, `rememberRuns`, `pullRequestFollowUp` and `Deps.PullRequestRecorded` drop the exception.
`pattern/PATTERN-landing.md` and `pattern/PATTERN-config-layers.md` held no mention of it.
`contracts/orchrole/role-beams.md:55-56` tells the orch that the follow-up rides only on a notice its `land_notice` allows.
The board holds `beam-pr-describes-itself` (T3), depending on this task, and the diff starts no work on it.

Section 2's six findings are already fixed at the merge base by `118bee8ad`, an ancestor of HEAD:
F26 in `internal/hubreconcile/reinterpret.go:103-123` (told code worktrees), F2 at `internal/dobbyengine/config.go:78`, `internal/configreg/configreg.go:147` and `internal/beamcli/mergeparent.go:86` (`beamsetting.Module`),
F27 at `internal/configreg/configreg_test.go:28`, F28 in `internal/configreg/runsettings_test.go` (the duplicated row is gone),
F5 behind the per-machine marker at `internal/hubreconcile/reinterpret.go:113-116`, and F13 at `internal/boardengine/store.go:291-296`.
The summary's claim to that effect holds, so leaving them is no scope gap.

The `Root-Cause:` trailer on `09d908924` names the defective condition the diff removes.
The test rows that gained `requested: true` keep their exact expected lines and follow from the new rule, so no check is loosened.
The removed rows pinned only membership behaviour.

## Rejected

- Assistant's rejected item "missing section 2 implementation": agreed, the fixes are at the review base.
- A PATTERN-config-layers reinterpretation for the `land_notice` change: the setting's documented meaning (on sends a line, off sends none) is unchanged; the diff removes a dobby override of it on the operator's decision, and no value could be rewritten to the old behaviour.
