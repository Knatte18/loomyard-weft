```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness over the PR-Review parent-ask diff. Each case below was run by reading the code.

**Cases run**

- `askParentReview`, record already asked: returns nil, no post. Correct.
- `askParentReview`, `ParentReviewWanted` or `AskParent` nil: returns nil, reads as off. Correct.
- `askParentReview`, `ParentReviewWanted` errors: logs a Warn, no ask, no record write, and the next entry retries. Correct.
- `askParentReview`, setting false: returns nil. Correct, and with the setting off `Call` behaves as before.
- `askParentReview`, `AskParent` errors: logs a Warn and writes no flag, so the next entry re-asks. Correct.
- `askParentReview`, `AskParent` returns `posted == false` (no orch recorded): logs at Info and writes no flag. Correct.
- `askParentReview`, post succeeds and `updateRecord` fails: returns an error, so the driver re-steps and the orch gets a duplicate mail (F2).
- Call site `openPullRequest`:
  - Runs after `ensureSession`, so `rec.StrandName` is the fresh or live session's name. Correct.
  - Not reached on the merged or closed recorded-PR path, the `ReviewEnded` path, or the `MergeAtOnce` path, which is right because no review is open there.
  - Runs once per pass before `wait`. Correct.
- `parentReviewRequest`: `Sprintf` with slug, `rec.URL`, task branch, parent branch and `rec.StrandName`. `StrandName` is set by `ensureSession` at `prsession.go:91`, so it is non-empty on this path. Correct.
- `pullrequest.Record.ParentReviewAsked` (`json:"parent_review_asked,omitempty"`): an older record reads as false, and `UpdateRecord` keeps unrelated fields. Correct.
- `recordPullRequest` with `hadRecord`: updates only number, URL, base, head and gated head, so the flag survives a re-entry. Correct.
- A review started afresh (the `ReviewEnded` record removed and a new head) writes a new record, so it asks again. This matches `doc.go`.
- `ParentReviewWanted` in `landingdeps.go`: `LoadConfig(fabricengine.ConfigDirs(l), "loom")` returns `cfg.PRReviewAskParent`. On error it returns a zero `Config`, so false plus the error, and the row logs it. Correct.
- `AskParent` in `landingdeps.go`: `orchcli.NotifyParent(l, loomProgram, line)` returns `(posted, err)`. It goes to the nearest live orch, and no orch gives `(false, nil)`. Correct.
- `loom.yaml` key: `pr_review_ask_parent: none`, so the local layer may set it, which fits the run_settings override. Correct.
- `envkit` fixture: `AskParent` returns `(false, nil)` and the setting reads false, so the fixture asks nothing. Correct.

**Findings**

1. **MEDIUM, design, `internal/landingshed/prreview.go:323-326` and `internal/landingshed/doc.go` (new lines)**
   - The board entry says the run waits at PR-Review until the orch answers satisfied or sends findings.
   - The code only mails the request once and goes into the unchanged `wait`.
   - That wait ends on merge, approval or the operator's `pr-continue`, never on the orch's answer.
   - Nothing records the orch's "satisfied", and `doc.go` states "The row's endings do not wait on the orch's answer".
   - The row waits only in the sense that it already waited on the operator. With the setting on, an operator who ends the review (or an approval) before the orch answers finishes the run with the orch's review unanswered.
   - Evidence: `prreview.go:323-326`, `judge` at `prreview.go:821-879`, `doc.go` new line "The row's endings do not wait on the orch's answer".

2. **LOW, design, `internal/landingshed/prreview.go:337-359`**
   - If the post succeeds and the `updateRecord` that sets `ParentReviewAsked` then fails, `askParentReview` returns an error and the driver re-steps.
   - The re-entry sees the flag unset and mails the orch the same request again.
   - The cost is one duplicate mail on a rare failure.
   - Fix: log the failed flag write like the other failures instead of returning it, or accept the duplicate and document it.

**Tool calls (3 of 6, one Bash call):** `grep` and `sed` of `LoadConfig`, `ensureSession` and `postNotice`/`loomProgram`. I ran them to confirm the config signature, that `StrandName` is set before the ask, and that `NotifyParent` returns `(false, nil)` when no orch is recorded. None of these was in my context.
