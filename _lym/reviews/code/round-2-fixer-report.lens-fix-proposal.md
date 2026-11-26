```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the F1 proposal that was carried out (read GitHub and run `foreignHeadHalt` before the entry push of a found, unended record).

For an open pull request on an unended review, I could not build a case where the proposed order is wrong:
- A pre-push status can only delay an approval ending by one poll.
- Bringing the hand commit into the task branch passes the check.
- API, status or text failures now stop before the push, which is harmless.

**Findings**

1. **MEDIUM: the new doc sentences claim more than the code does.**
   - **Changed text:**
     - `PRReview` doc: "A re-entry on a review not ended halts before any push when the branch on GitHub holds a commit pushed by hand."
     - `openPullRequest` doc: "a hand commit on its branch on GitHub halts with nothing pushed".
     - `docs/overview.md`: "halts the row, at a re-entry before any push".
   - **Evidence:** the entry check runs only under `if status.State == pullrequest.StateOpen`. A closed or merged status with `head == rec.GatedHead` still reaches `pushTask`.
   - **Concrete case:** the row halted on a closed pull request, the operator pushes a fix to the branch, then runs `lym loom resume` before reopening. The push runs with no check. By the parent's reading of `resetUnnamedTip`, it can reset the hand commit off the branch.
   - **Fix:** qualify all three sentences with "while GitHub reports the pull request open".

2. **MEDIUM: the proposal's "not `ReviewEnded`" scope leaves part of F1's premise in place.**
   - **Changed text:** `if !found || rec.ReviewEnded { pushTask(...) }`.
   - **Evidence:** F1 lists a resume after a dirty-tree halt. `dirtyEndReason` is returned for ended reviews (`judge`, the `rec.ReviewEnded` branches).
   - **Concrete case:** the operator ends the review, the row halts on a dirty tree, a commit is then pushed to the branch on GitHub, and the operator resumes. With `head == rec.GatedHead`, the row pushes before it reads GitHub. The push is rejected, or resets the hand commit off the branch, and `Finalize` then merges without it.
   - **Also affected:** an ended review whose head moved removes the record and pushes the same way.
   - This behaviour predates the fix, which did not make it worse. Record it in the report if it is not changed.

Outside my lens: `gateHead`'s push has no re-read after the gate, so a hand commit pushed while a long gate runs would meet that same push.

Tool calls made: none.
