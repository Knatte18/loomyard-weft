# Support log — archive-tag-before-dirty-check

## Interview

The session ran autonomously; no operator answered.
Each turn below is a question the exploration raised and the pick made for it.

1. **What does the task ask?**
   The board entry (id 28) says `Remove` pushes the archive tag before the no-force dirty checks, so a dirty-sibling refusal pushes a tag missing the uncommitted records and the resume pushes a second one.
   Its instruction: move the archive after the dirty checks, still before any removal.
2. **Can the archive sit after the dirty checks and before every removal with the current order?**
   No.
   The current order is refusals → archive → portal and launcher teardown → dirty checks → removals.
   The portal and launcher teardown sits between the archive and the dirty checks, so either the dirty checks move ahead of that teardown or the archive lands after a removal.
   Pick: move the dirty checks ahead of the portal and launcher teardown (D1).
3. **Is there a reason the portal teardown precedes the dirty checks?**
   The file header says it lets the teardown run when the worktree dir is already gone.
   `git log -S` traces that to `9de0c46d6`, where the portal teardown preceded the target-exists check.
   It now follows that check, and a missing dir returns "worktree not found" first, so the rationale is stale.
   The `4b070e31e` hardening pass added `nameStrandedPortalTeardown` to name the reconcile remedy precisely because it chose not to reorder, citing the same stale header.
   Pick: reorder and delete the remedy helper (D2, D3).
4. **Do the other `archiveWeftTip` callers have the same defect?**
   `Cleanup` skips protected and checked-out branches before archiving, and `RemovePairBranch` refuses a present sibling before archiving; neither has a dirtiness gate after its archive.
   Pick: out of scope.
5. **Does `battencli`'s refusal wording depend on the old order?**
   `teardownRefusal` names `commit-records` then resume for `ErrPairSiblingDirty`, and a plain resume for `ErrArchiveFailed`; both stay correct.
   Pick: no change.
6. **Which tests pin the old order?**
   `TestRemove_ResumedRemoveReusesArchiveTag` asserts a tag exists after a dirty refusal;
   `TestRemove_RefusalNamesStrandedPortalTeardown` and `TestRemove_RefusalWithNothingStrandedOmitsRemedy` assert the stranded-portal remedy.
   `archiveWeftTip`'s same-tip reuse path has no other test.
   Pick: D6.

## Rejected alternatives

- **Archive between the dirty checks and the junction sweep, portal teardown left where it is.**
  Puts the archive after a removal, against the task's "still before any removal", and a failed archive would then strand the portal and launchers, breaking the documented "a failed archive leaves everything in place" rule.
- **Keep `nameStrandedPortalTeardown` for defence in depth.**
  Nothing is recorded before the dirty checks after the reorder, so its only live branch is unreachable; it would be dead code.
- **Make `archiveWeftTip` delete or supersede an earlier tag of the same slug on the resume.**
  Treats the symptom, deletes an already-published ref, and leaves the tag missing the records pushed in the first place.
- **Drop the reuse test along with the old resume test.**
  The reuse path is part of `archiveWeftTip`'s contract (a resumed teardown on the same tip finds its own tag) and has no other coverage.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 — Does "before any removal" include the portal and launcher teardown? — Resolved, auto-pick: yes; the dirty checks move ahead of it (D1).
- Q2 — Is the portal-before-dirty-check order still justified? — Resolved, auto-pick: no; the rationale predates the target-exists check moving ahead of it (D2).
- Q3 — Delete the stranded-teardown remedy helper in this task? — Resolved, auto-pick: yes, it becomes dead code (D3).
- Q4 — Fix `Cleanup` and `RemovePairBranch` too? — Resolved, auto-pick: no; neither has a refusal after its archive.
- Q5 — Change `battencli`'s `teardownRefusal` wording? — Resolved, auto-pick: no; both remedies stay correct.
- Q6 — Clean up duplicate archive tags already pushed by the old order? — Resolved, auto-pick: no; out of scope and nobody asked for it.
- Q7 — Keep a test for `archiveWeftTip`'s same-tip reuse? — Resolved, auto-pick: yes, as a pre-seeded same-tip tag (D6).
- Q8 — Do the livestate matrix or mutation-oracle tests encode the old refusal records? — Resolved in review round 1: yes; `livestate_verbs_test.go`'s `removeCase` dirty cells and `mutation_record_integration_test.go`'s `Remove` dirty-refusal test assert the portal and launcher teardown before the refusal, and invert (D6).
  No `internal/battencli` integration test drives a dirty refusal, so the battencli risk was dropped.
- Q9 — Keep the `9de0c46d6` archaeology in D2? — Resolved in review round 1: no; the decision record keeps only the stale-rationale fact, and Interview turn 3 keeps the history.
