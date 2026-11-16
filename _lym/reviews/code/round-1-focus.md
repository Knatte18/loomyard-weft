---
round: 1
focus:
  - "internal/battencli/startguard.go `refuseProject`: with the filing mark set, the function returns judged=false, so the task checks run next. Settle whether those checks (dirty source, left pair, the resume-past-Worktree-Create pass) behave correctly for a filed arras run whose entry is now a grouping project, and whether any path still refuses it or wrongly lets it through."
  - "The `found && g.FilingMarked != nil` condition: settle what `found` means here (run on record) and whether a filing mark on a slug with no run record, or a stale mark, could let a grouping project start fresh. Check the mark is read for the slug and not for a parent or sibling."
  - "Error path of `FilingMarked`: settle whether `probeRefusal` yields a refusal with the way-forward text the neighbouring probe errors use, and whether a read error is correctly judged=true rather than falling through to the grouping-project refusal."
  - "`hubStartGuard` wiring: settle that `shedrun.FilingMarked(location, slug)` has the signature and semantics the guard comment states, and that every other construction of `startGuard` (tests, other callers) is safe with the new nil-able field."
  - "Scope against the board entry: the entry offers two ideas, let the guard pass or have filing finish the run. Settle that the diff makes the first fix, covers every non-beam grouping project and not only beam, and leaves `cleanup.go` `ClearMotifRun` consistent so the done run reaches Land and Worktree-Teardown."
  - "internal/battencli/startguard_test.go `TestStartGuard_Refuse`: settle that the new cases cover the filed-run pass, the unmarked grouping-project refusal, and the probe-error refusal without duplicating existing cases."
---

Seed focus for round 1.
The diff is two files, the start guard and its test, so the focus is on the guard's control flow and its callers.
