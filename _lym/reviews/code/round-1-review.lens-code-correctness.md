```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. For each function and branch the diff changes, I ran one concrete case and marked it correct or not.

**Cases run**

- `ResolvePullRequest`: all nine parent/field pairs. Correct. A missing fact under parent `on` with field empty is a no. Field `on` or parent `required` halts, and the cancel hint appears only for field `on` under a non-required parent. A github.com origin that `ParseOwnerRepo` refuses always halts. Correct.
- `originHost`: scp form, `ssh://`, user:pass@, and a local path. Correct.
- `PRReview.Call` and `decide`: an unreadable beam.yaml is Stuck/Halt. The no and yes paths dispatch correctly. Defect F1 on the no path.
- `closeRecordedPullRequest`: the record is closed only when GitHub reports it open, then removed. Correct, except for the session leak in F1.
- `openPullRequest`: first entry, re-entry with an open record, record with `ClosedForContinue` at the gated head, and record with `ClosedForContinue` at a moved head. All correct. The moved-head case removes the record and opens a fresh pull request. The closed, not-merged record starts a session just to halt (F5).
- `holdWhileCovered`: pause covers, then lifts. It stops a live session once, resumes only a session it stopped, and calls no GitHub, push or gate while covered. Correct. The unreadable pause record is fail-open (F6).
- `wait`, `pollStatus`, `nextPRWait`: the first wake with no entry status polls at once, and with a fresh entry status it reuses it. The continue flag appearing polls at once. A transient failure logs one Warn per episode and keeps waiting. The interval stays between 1s and 1m and GitHub is polled at most once a minute. Correct.
- `wake`: defect F2, because the gate runs before the status is read.
- `gateHead`: clean tree, moved head, a pass pushes and records. A head or tree that changed during the gate discards the verdict. A failure is mailed once per head. Correct, apart from F4 and a small race between the after-gate check and `PushTask`.
- `judge`: merged with missing commits, a head other than the gated one, or a dirty tree; closed; open+approved; open+`Continue`. Correct for the cases that reach it, subject to F2. The continue path stops, re-reads, closes and records. A failed `Close` goes through `pollFailure`. A failed record write after a successful close recovers on re-entry, because closed plus `Continue` plus clean plus gated gives `ClosedForContinue`.
- `standingReviews` and `Status`: latest of `APPROVED`, `CHANGES_REQUESTED` or `DISMISSED` per login. `COMMENTED` and `PENDING` are ignored. `Approved` needs the review's commit id to equal the head. Merged beats closed. Correct.
- `ensureSession`: the live strand is reused, a recorded session id is resumed when its strand is not live, and with none recorded a new session starts. Defect F7 on a failed record write.
- `stopLiveSession`, `endSession`: a cancelled context leaves the session running. Correct, except F1.
- `Finalize.pullRequestLanding`: `ParentMode` nil lands locally. The no with a record closes and removes it. A yes with no record halts behind `LandingStopPrefix`. Open sets merge. Closed with `ClosedForContinue` lands locally, and closed without it halts. Merged lands through the already-landed path. Correct.
- `mergeOnGitHub`: the push, then merge with the pushed head as expected head. A pending refusal is retried for under a minute. A method refusal names `squash`. Other refusals halt with their way forward. Fetch and fast-forward failures halt. Correct.
- `pairteardown.closeOpenPullRequest` and `closePairPullRequest`: a gone worktree does not stop the teardown. Merged or closed pull requests are left alone. A failed client, status or close is a Warn and the teardown goes on. `EndForCancel` and `RemovePair` both call it; the second call finds the pull request closed and does nothing. Correct, except F3.
- `prContinueVerb`: an absent record is refused, repeat calls are idempotent, and a record removed mid-write maps to the no-record message. Correct.
- `childAtWaitingRow`: counts `running` at `PR-Review` as a live wait. Correct.
- `StartSession`, `allOutputFilesExist`, `stopOutcome`: an unawaited run's empty output list reads as not finished, so stop records `stopped`. Correct.

**Findings**

- **F1, LOW, design** — `internal/landingshed/prreview.go:53-67`, `:144-173`, `internal/landingshed/prsession.go:216-223`. The no path removes the pull-request record before `Call` runs `endSession`. `stopRecordedSession` then finds no record and stops nothing. A session left live by an interrupted earlier call (context cancelled, re-entry, mode now no) is orphaned. D7 says Done stops the session. Fix: stop the recorded session in `closeRecordedPullRequest` before `RemoveRecord`.
- **F2, MEDIUM, design** — `internal/landingshed/prreview.go:414-446`, `:616-617`. `wake` runs the gate and push before it reads the pull request's status. Take a PR merged on GitHub at an ungated head M, which an agent commit carried by the detached push produces. If the task head equals M and the gate passes, the wake pushes and sets `GatedHead = M`. `judge` then sees `status.HeadSHA == rec.GatedHead` and ends Done. D9's "merged with a head other than the record's gated head" halt never fires, and the row also gates and pushes onto an already merged or closed pull request. Fix: poll the status first on a wake, and skip `gateHead` when the state is merged or closed. Alternatively, compare the merged head against the gated head the record held before the wake's gate.
- **F3, LOW, consistency** — `internal/pairteardown/pullrequest.go:75-77`. D12 says a gone task worktree logs a Warn naming the task branch. The Warn carries only `slug` and the generic phrase "its task branch". It also fires on every gone-worktree teardown, including runs that never opened a pull request. Fix: include the task branch name as an attribute, and word it so it does not claim a pull request exists.
- **F4, LOW, design** — `internal/landingshed/prreview.go:414`, `:513-515`. A head with `MailedHead == head` is never gated again, even when its failure was environmental (timeout or gate could not start). Such a head stays ungated until a new commit, which blocks approval-based Done. D8 asks only that the mail go once per failing head. Fix: record the failure and mail once, but re-gate the head after a long backoff, or after an infrastructure-class failure.
- **F5, LOW, design** — `internal/landingshed/prreview.go:231-238`. A recorded pull request found closed without a merge, with no `Continue` flag, calls `ensureSession` and then ends Stuck. `Call` immediately stops the new session. Fix: start a session on that branch only when `rec.Continue` is set.
- **F6, LOW, consistency** — `internal/landingshed/prreview.go:315-327`. The doc comment says a `PauseCovered` error is logged once, but the Warn fires on every read. The error is also read as uncovered, so the row can gate, push and poll while a pause record is unreadable. Fix: log once per hold, or fail closed, and make the comment match.
- **F7, LOW, design** — `internal/landingshed/prsession.go:80-94`. If `StartSession` succeeds and `updateRecord` fails, the call errors with the new session live and unrecorded. The re-entry finds no recorded strand and starts a second session for the run. Fix: stop the new strand when the record write fails.
- **F8, LOW, design** — `internal/landingshed/prreview.go:437-449`, `:457-465`. With `rec.Continue` set and the tree dirty or ungated, every poll stops and resumes the session while the agent is cleaning up, about once a minute. D9 stops the session only for a merged pull request or an approval. Fix: skip the stop when the tree is dirty, since `judge` waits anyway. Alternatively, accept it and say so in the doc comment.

**Tool calls**

1. `cat internal/pullrequest/client.go` — the `Status` and review semantics were not in context.
2. `cat internal/pairteardown/pullrequest.go`, the diff of `teardown.go` and `cancel.go`, and `cat internal/loomcli/prcontinue.go` — none of them were in context.
3. `git diff` of `finalize.go`, `verifygate.go`, `battencli/wire.go` and the shuttle `run.go`, `wait.go` and `stop.go` — their changes were not in context.
