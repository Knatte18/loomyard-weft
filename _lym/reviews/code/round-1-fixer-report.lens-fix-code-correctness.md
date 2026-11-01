```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the code correctness of the fix's changed branches (`AcceptedHead`, `AcceptFixReview`, `judgeFixerChange` and `gateAndFix`) and the tests it added.

**Findings**

1. **LOW**: `internal/landingshed/finalize.go`, the `judgeFixerChange` reset (`if rec.AcceptedHead != "" { rec.AcceptedHead, rec.ReviewRounds, rec.Findings = "", 0, nil }`).
   - **Summary:** No test fails if this reset is removed, and the first version of it (unguarded) crashed the tests only by accident.
   - **Evidence:**
     - The new test "a fixer change made after the operator's acceptance is reviewed" ends in an approval, which returns before the round-cap check. Without the reset it would still land.
     - It would take a rejection after a stale acceptance to show that `ReviewRounds` carried over (2, then 3, so an immediate `haltReviewCapped` with no fixer turn). No row covers that.
     - Its title ("whatever the caps the earlier rounds spent") promises more than it asserts.

2. **LOW**: the `gateAndFix` doc comment ("lands the change when the operator accepted it") is stale.
   - **Summary:** It should say the change is landed only when accepted at the current HEAD.
   - **Evidence:** the comment predates `AcceptedHead`, and the `judgeFixerChange` comment was updated but this one was not.

3. **LOW**: behavioural note on the merge-then-compare design, not a defect in the fix as written.
   - **Summary:** If the parent moves between accept and resume, `mergeParent` makes a merge commit, so HEAD no longer equals `AcceptedHead`.
   - **Evidence:**
     - The operator's acceptance is then discarded and an unchanged over-cap fixer change is sent back to the fix review.
     - That can halt the run for acceptance again.
     - `Reviewed` has the same property, so this is consistent. I found no test or doc that states it.

**Checked and holding**
- **Signature change:** `AcceptFixReview` has a single production caller, `loomcli/circling.go`, and it was updated. The test fake in `circling_test.go` wraps `acceptFinalize`, which is unchanged.
- **`AcceptFixReview` paths:**
  - A failing `taskHead` returns before any write, and the new test covers it.
  - The retry path (heading present, `AcceptedHead` empty) appends nothing and records the head.
  - A second call returns early once `AcceptedHead != ""`.
- **Accepted resume:** `gateAndFix` keeps the rounds, and `judgeFixerChange` proceeds when `AcceptedHead == head`.
- **Stale acceptance:** it is reset and reviewed (I traced this by hand). The "earlier HEAD e9" row and the failing-then-passing row would both fail with the old flag semantics, because no review session would run.
- **Pending-findings rows:** `runFixer` clears `FixReview` and writes it before `judgeFixerChange`. The start-failure row asserts the record on disk holds an empty `FixReview`, which fails if that write is removed. I did not confirm that `fixerRun.run` persists the record with `FixReview` still set, so that last step rests on the code as I read it.
- **Other tests:** the F6 renames and the F4/F8 test edits are consistent with their predicates.

**Further tool calls:** none. My context already held every file I needed.
