```yaml
lens: fix-code-correctness
outcome: reported
```

Checked the correctness of the code the fix changed (`openPullRequest` entry order, the comments on it, and the changed `TestPRReview_Wait` rows).

**MEDIUM**
- **Changed text:** the `openPullRequest` doc ("For a recorded review that has not ended, … a hand commit on its branch on GitHub halts with nothing pushed") and the `PRReview` doc ("A re-entry on a review not ended halts before any push when the branch on GitHub holds a commit pushed by hand").
- **Summary:** both claims hold only when GitHub reports the pull request open.
- **Evidence:** the new check is gated on `status.State == pullrequest.StateOpen`. A closed or merged status with `head == rec.GatedHead` falls through to `pushTask` with no check.
- **Concrete case:** the row halts on a closed pull request, the operator pushes a fix to the branch on GitHub without reopening it, then runs `lym loom resume`. The entry reads closed and pushes. By the `resetUnnamedTip` code, that push can reset the branch and drop the commit.
- **Scope:** the behaviour for closed and merged pull requests predates this fix and follows the review's proposed "when that status is open". The wording is what the fix added. The fix is to say "open pull request" in both comments, and likewise in the `docs/overview.md` sentences.

No other findings:
- **Entry order:** I traced these inputs through the new order and each behaves correctly:
  - no record
  - ended review at the gated head and past it
  - unended review with an ungated head, an empty `GatedHead`, or the gated head
  - a failure of the status read, the API open or the push
- **Entry-halt rows:** the two `entryHalt` rows fail with the old push-first order, because the push count becomes 1.
- **Moved-head row:** it fails if the check in `wake` is removed, because the gate runs.
- **Continued-review row:** it fails if `judge` ends a continued review before its foreign-head check.

Outside the fix, noted only: `gateHead`'s push after a long gate, and the early push for an ended review whose head moved, can still drop a hand commit made on GitHub.

Further tool calls: none.
