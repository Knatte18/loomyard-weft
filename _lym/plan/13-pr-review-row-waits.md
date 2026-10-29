# Card 13 — pr-review-row-waits

**Create:**
- `plan:internal/landingshed#PRReview.wait` -> `func (p *PRReview) wait(ctx context.Context, api pullrequest.API, rec pullrequest.Record) (shedengine.Outcome, shedengine.OutputPointer, error)`
- `plan:internal/landingshed#PRReview.holdWhileCovered` -> `func (p *PRReview) holdWhileCovered(ctx context.Context) error`
- `plan:internal/landingshed#nextPRWait` -> `func nextPRWait(current time.Duration, moved bool) time.Duration`
- `plan:internal/landingshed#prReviewTestDeps` -> `func prReviewTestDeps(t *testing.T, api *fakePullRequestAPI) Deps`
- `plan:internal/landingshed#TestPRReview_Wait` -> `func TestPRReview_Wait(t *testing.T)`

**Edit:**
- `plan:internal/landingshed#PullRequestDeps`
- `plan:internal/landingshed#fakePullRequestAPI`
- `internal/landingshed#PRReview`
- `internal/landingshed#NewPRReview`
- `internal/landingshed#PRReview.Call`
- `internal/landingshed#TestPRReview_Call`
- `internal/landingshed/doc.go`
- `internal/loomcli#landingDeps`
- `internal/testkit/envkit#LandingDeps`
- `docs/overview.md`

**Uses:**
- `plan:internal/pullrequest#Status`
- `plan:internal/pullrequest#UpdateRecord`
- `internal/landingshed#verifyGate`
- `internal/landingshed#newVerifyGate`
- `internal/landingshed#verifyGate.check`
- `internal/landingshed#fakeVerifyTree`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/logcapture#CaptureVerbose`
- `internal/loomcli#loomProgram`
- `internal/loomcli#pauseCovered`
- `internal/loomcli/doc.go`

**Intent:**
D8, D9, the operator's continue ending and D10's wait mark: after recording the pull request (card 11), the row's call waits on it in process, as pawl's Beam-Await does, and takes no parent lock on any path.

`PullRequestDeps` gains `CommitsMissing func(merged, task string) ([]string, error)` (task commits not reachable from merged), `DirtyPaths func() ([]string, error)` (the task worktree's uncommitted paths), `MailAgent func(strandName, line string) error`, `PauseCovered func() (bool, error)`, `Now func() time.Time` and `Sleep func(ctx context.Context, d time.Duration) error`.
Card 11's fail-loud wiring guard in `Call` covers each of them too, so a nil one is an error naming the field, never a nil call.
`PRReview` gains a `gate verifyGate`, built by `NewPRReview` through `newVerifyGate(deps)` with its `dirty` seam set to `DirtyPaths`, whose key is the entry's `pr-review`.
`Call` ends a yes with `wait` instead of Done.

A covering pause: `holdWhileCovered` reads `PauseCovered` and, while it reads covered, sleeps through `Sleep` and reads again, returning once uncovered or on a cancelled context; a `PauseCovered` error is logged once and read as uncovered, as loomcli's `pauseCovered` reads an unreadable record.
`Call` runs it before card 11's push, so an entry made under a pause pushes nothing and calls no GitHub until the pause lifts; `wait` runs the same read at every wake, and a covered wake does no gate, push or poll and keeps waiting (the departure in the overview's Shared Decisions).

`wait` sets the driver strand's mark through `Deps.VerifyWaitMark` as `PR #<n>` at its start and again after each gate, whose own verify mark replaces and clears it, and clears it on every way out; a mark error is logged and changes nothing.
Each wake, at least a second apart, doubling to a minute while nothing changes and back to a second when the task head moves (`nextPRWait`, as `nextAwaitWait`):
1. a cancelled context is an error (`cancelErr`); a covering pause keeps waiting as above;
2. it reads the task head and `DirtyPaths`; a dirty tree is the agent at work, and the wake goes on to the poll without gating;
3. a clean tree whose head differs from the record's `GatedHead` runs `gate.check`; afterwards it re-reads head and tree, and a changed head or a dirty tree discards the verdict with no halt; a counted pass pushes with `PushTask` only while the head still equals the gated head, then records it as `GatedHead`; a counted failure pushes nothing and, once per failing head (`MailedHead`), posts the gate's reason, which names the failing tier and log path, to the record's `StrandName` through `MailAgent`, or logs it when no strand is recorded; a failed post is logged at Warn and the head counts as mailed;
4. at most once a minute, and at once when the record carries `Continue`, it reads `Status`.
A failed `Status` is judged as the overview's GitHub failures: `ErrTokenUnresolvable`, even wrapped in a `*url.Error`, is the halt naming `gh auth login --hostname github.com`, then `lym loom resume`; a transient failure logs one Warn per episode and keeps waiting; any other failure is Stuck with Halt naming the error and the token and repository access, then `lym loom resume`.
The poll's verdicts, each re-reading head and tree first:
- merged, with task commits the merged head lacks (`CommitsMissing(status.HeadSHA, taskHead)` non-empty; a `CommitsMissing` error is an error stop of the row): Stuck with Halt naming them; way forward: file them as a board note, then `lym batten cancel <slug>`, then `lym board set-status <slug> done`, since the pull request's work is on the parent while the cancel leaves the entry cleared;
- merged, with a merged head other than `GatedHead`: Stuck with Halt naming both heads; way forward: `lym loom goto Finalize`, which fast-forwards the local parent to the merge under the lock and lands through the already-landed path, marking the task done, then run the gate over the parent branch in its worktree and file a board note for any fix it needs;
- merged, with the merged head equal to `GatedHead` and the task head: Done on a clean tree; on a dirty tree Stuck with Halt naming the uncommitted files; way forward: discard them in the task worktree, after filing any worth keeping as a board note, then `lym loom resume`;
- closed and not merged: Done when the record carries `ClosedForContinue` or `Continue`, recording `ClosedForContinue` when it was missing, since a close lym made for the continue flag whose record write failed reads the same; otherwise Stuck with Halt naming the pull request; way forward: reopen it on GitHub and run `lym loom resume`, or `lym batten cancel <slug>`;
- open, with `Continue` in the record: on a clean tree whose head equals `GatedHead`, it closes the pull request with the comment `the operator ended the review through the PR-Review agent; lym closes this pull request and the run lands through Finalize`, records `ClosedForContinue`, and is Done; a dirty tree or an ungated head keeps waiting, and step 3 gates a moved head first; a failed close is judged as a failed `Status`;
- open, `Approved` and not `ChangesRequested`, with `HeadSHA` equal to `GatedHead` and the task head: Done when the re-read head is unchanged and the tree clean, otherwise keep waiting;
- open otherwise: keep waiting.
A merged pull request takes the merged verdicts whatever the record's `Continue` says, since nothing is left to close.
Bound: the wait has no timeout; it lasts until the pull request is approved, merged or closed, the operator continues, or the run is cancelled, holding one of the parent dobby's `max_runs` slots meanwhile.

`prReviewTestDeps` builds a `Deps` whose `PullRequest` seams are all filled with scriptable fakes over a temp record and the given API fake, which `TestPRReview_Call`'s rows switch to, so no row hits a nil seam.
`TestPRReview_Wait` (a new top-level test, since `TestPRReview_Call` covers the entry and this covers the loop; it stays serial, since it captures the process-global logger) runs `Call` to the wait over `fakePullRequestAPI`, `fakeVerifyTree`, scripted heads and dirty paths and an injected clock and sleep, table-driven, a row per verdict above plus: a dirty tree waits without gating; a moved clean head is gated and pushed on a pass and recorded as `GatedHead`; a failing gate pushes nothing and mails once per head over two wakes; a failing mail post is logged and not retried for that head; a head moved during the gate, and a tree dirtied during it, each discard the verdict; an approval of an older head and a comment after a change request keep waiting; a transient poll error logs once over two failing polls and keeps waiting; an unresolvable token wrapped in a `*url.Error` halts naming `gh auth login`; a 403 halts naming the token and repository access; GitHub is polled at most once a minute of the injected clock; the wake intervals stay between a second and a minute; a covering pause at a wake skips the gate and the poll and the wait goes on once uncovered; a covering pause at entry pushes nothing and calls no GitHub until uncovered; the continue flag closes the pull request and is Done, with a dirty tree keeps waiting, and with a merged pull request takes the merged verdict and closes nothing; a closed pull request whose record carries `Continue` but not `ClosedForContinue` is Done; a nil new seam is an error naming it; the mark is set at the start and after a gate and cleared on Done, a halt and an error; and a `Deps.Lock` store holds no record for the parent after every row, which is the store the shared guard would take a lock through.
`fakePullRequestAPI` gains a counter of `Status` calls, which the once-a-minute row reads.
`TestPRReview_Call`'s yes rows now script a `Status` that ends the wait.
loomcli `landingDeps` fills `CommitsMissing` through the task pair's `CommitLog(merged, task)`, `DirtyPaths` through `verifytree.DirtyPaths` over the task worktree, `MailAgent` through `creelcli.NewProgramClient(l, loomProgram)`'s `Post`, `PauseCovered` through the run's pause record (as `pauseCovered` reads it), `Now` with `time.Now` and `Sleep` with a context-aware sleep; envkit's `LandingDeps` stubs the new seams.
The landingshed doc's PR-Review paragraph and `docs/overview.md` describe the wait, its endings, the continue flag, the pause hold and the mark.

**ImpactSummary:** Turns PR-Review's yes from an immediate Done into a lock-free wait inside the same producer; new seams are filled by loomcli's wiring and envkit, and no other row changes.

**Commit:** 13: PR-Review waits on the pull request, gates agent commits and ends on its state
