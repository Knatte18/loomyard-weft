```yaml
lens: fix-way-forward
outcome: reported
```

Task: check, through the fix-way-forward lens, that each way forward the F12 fix introduced can be reached and makes progress.

No findings.

- **Clean-tree reroute clears the count.** This works. It sits in the same step the record already defines, where the wake clears the mark, `parent_review_asked` and the session id and returns a reroute. That step is reached on a clean tree, and on the resume after the dirty-tree Stuck.
- **"A re-step, a crash-resume and the resume after the dirty-tree Stuck keep it."** Each of these can be reached:
  - A re-step comes from a transient push, GitHub or session-start error, which `PRReview.Call` returns as an error for the driver to re-step.
  - A crash-resume goes through `lym loom resume`.
  - The dirty-tree Stuck names commit or discard, then `lym loom resume`.

  None of them clears the count, so the cap carries over each one.
- **Criterion "re-entered after a send-back starts with a fix-here count of zero, and a re-step of the row keeps the count".** A test can reach both halves: a send-back, the reroute, then the run going down to PR-Review again; and a re-step forced by a transient error.
- **Bound "between two send-backs, or before the first".** Each send-back's clear starts a new allowance. The reroute cap still bounds how many send-backs can happen, so this always ends in either a send-back or a cap halt.

Clearing the count at the reroute-cap halt resembles the review's rejected item F-D, which is about the mark. The fix did not cause it and does not make it worse. After the cap halt, the resume starts with a zero count, and each later cap halt needs an operator resume, so the run still makes progress.

Further tool calls: none.
