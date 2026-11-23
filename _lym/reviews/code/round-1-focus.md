---
round: 1
focus:
  - "Scope: the decision record's Goal has section 1 (member line) and section 2 (F26, F2, F27, F28, F5, F13). The range's diff touches only the landing-notice files and `contracts/orchrole/role-beams.md`. Settle for each of F26, F2, F27, F28, F5 and F13 whether the diff lands it, or whether the Goal's own wording (F5 and F13 only matter while a hub holds the retired board fields) or an `Added after Discussion` entry licenses leaving it out."
  - "`internal/dobbyengine/landnotice.go` `rememberRuns`: with `Member` and the `members` map gone, does a landed line go out exactly when the remembered reading is on, with the follow-up kept on that line? Check an older state record that still carries a `member` field loads and behaves, and that nothing else still reads `LandNoticeState.Member`, `isBeamMember` or `landNoticeTasks`."
  - "Leftover mentions of the exception: grep `pattern/PATTERN-landing.md`, `pattern/PATTERN-config-layers.md`, `contracts/`, `internal/dobbyengine` docs and stencils for 'whatever', 'member' and 'land_notice' wording that still says a member gets its line regardless of `land_notice`."
  - "`contracts/orchrole/role-beams.md`: does the reworded passage tell the orch what the code now does, including that the follow-up rides only on a landed line a member's `land_notice` allows?"
  - "`TestLandNotices` in `internal/dobbyengine/landnotice_test.go`: do the member rows still check what their names say after the change (the off-member row sends nothing under a recorded pull request, the on-member row carries the follow-up)? Judge whether removed rows (failed board read, non-member entry, done entry) tested only the removed membership behaviour or also pinned something still live, and whether the commit's `Root-Cause:` trailer names the defect the fix removes."
  - "Loosened checks: rows that gained `requested: true` or lost `want` lines; confirm each change follows from the new rule and no assertion was widened without a `Loosened-Check:` trailer."
  - "Note `beam-pr-describes-itself`: the Goal asks for the follow-up to be noted as out of scope; check whether the diff or a record carries that note, and that the diff does not start on it."
---

Seed focus from the decision record and the range diff.
The diff makes the section 1 change; section 2's findings show no sign of landing in it, which is the first question for the reviewer.
