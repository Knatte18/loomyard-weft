# Card 15 — finalize-through-pull-request

**Create:**
- `plan:internal/landingshed#Finalize.pullRequestLanding` -> `func (fz *Finalize) pullRequestLanding(ctx context.Context) (prLanding, shedengine.Outcome, shedengine.OutputPointer, error, bool)`
- `plan:internal/landingshed#prLanding` -> `type prLanding struct`
- `plan:internal/landingshed#Finalize.mergeOnGitHub` -> `func (fz *Finalize) mergeOnGitHub(ctx context.Context, parentHandle parentMerger, landing prLanding, title, message string) (shedengine.Outcome, shedengine.OutputPointer, error, bool)`
- `plan:internal/landingshed#TestFinalize_PullRequest` -> `func TestFinalize_PullRequest(t *testing.T)`

**Edit:**
- `internal/landingshed#Finalize.Call`
- `internal/landingshed#Finalize.pushParent`
- `internal/landingshed/doc.go`
- `PATTERN.md`

**Uses:**
- `internal/landingshed#ResolvePullRequest`
- `plan:internal/landingshed#PullRequestDeps`
- `plan:internal/landingshed#fakePullRequestAPI`
- `internal/pullrequest#MergeRefusal`
- `internal/pullrequest#RefusalKind`
- `internal/pullrequest#ReadRecord`
- `internal/pullrequest#RemoveRecord`
- `internal/landingshed#recordingParentMerger`
- `internal/landingshed#LandingStopPrefix`
- `internal/landingshed#Finalize.halt`
- `internal/landingshed#TestFinalize_MergeMarkPushOrderAndVerdict`
- `internal/landingshed#TestFinalize_LockFirstAndFastForward`

**Intent:**
D11: Finalize lands through the pull request, and merging it moves the parent, so it happens only here, under the parent lock (`PATTERN-parent-lock`).
`pullRequestLanding` runs as Finalize's first step under the lock, right after `guard.ensure`, and follows the resolver, never the record alone: it reads `PullRequest.ParentMode`, `EntryField` and `Origin` exactly as PR-Review does (an unreadable `beam.yaml` halts with the same reason, behind `LandingStopPrefix`) and calls `ResolvePullRequest`; a resolver `Halt` is Stuck with Halt behind `LandingStopPrefix`.
Every GitHub call here is judged as the overview's GitHub failures: `ErrTokenUnresolvable` first, even wrapped in a `*url.Error`, halts naming `gh auth login --hostname github.com`, then `lym loom resume`; a transient failure is an error through `transientFailure`; any other failure halts naming the token and repository access, then `lym loom resume`; each halt opens with `LandingStopPrefix`.
- A no closes a recorded pull request that `Status` reads open, with the comment `lym closed this pull request: <reason>`, removes the record (a merged or closed one is left as it is), and Finalize lands locally as today.
- A yes whose record is absent is Stuck with Halt behind `LandingStopPrefix`: `this landing goes through a pull request (<reason>) but the run has no pull request recorded; way forward: lym loom goto PR-Review, which opens it`.
- A yes whose record names a pull request GitHub reports closed while the record does not carry `ClosedForContinue`, `Continue` alone included, is Stuck with Halt behind `LandingStopPrefix`: `the run's pull request <url> was closed without a merge; way forward: reopen it on GitHub, then lym loom goto PR-Review, or lym batten cancel <slug>`; only PR-Review records `ClosedForContinue`, after its clean-tree and gated-head checks.
- A yes whose record carries `ClosedForContinue` with a closed pull request is the operator's ending: Finalize lands locally as today, with its gate and trailer.
- A yes whose pull request GitHub reports merged lands as today: the parent sync fast-forwards to the merged upstream and the parent-side merge takes the already-landed path.
- A yes whose pull request is open marks the landing to merge on GitHub.
A Finalize built with a nil `PullRequest.ParentMode`, as the existing tests and struct-literal builds make it, skips the step and lands locally, following the package's nil-is-absent seam convention; loomcli's `landingDeps` always fills it and its drift guard keeps it filled.
Finalize then runs as today through the parent sync, the merge of the parent into the task worktree, the fixer and the `finalize` gate, the parent-records commit and the lock re-check.
For a landing marked to merge on GitHub, `mergeOnGitHub` runs at step 6, after the lock re-check and before the parent-side `Merge` call: it pushes the task branch with `PushTask` (a failure judged as PR-Review's push failure, the halt opening with `LandingStopPrefix`) and reads the pushed head, merges through `API.Merge` with that head as the expected head, `Config.Squash` choosing squash or merge, the change description's title as the commit title and its body with the trailer block as the commit message; a `MergeRefusal` of kind `RefusalPending` is retried with `Sleep` for at most a minute of `Now` under the lock.
It then reads `ReadParentTips(true)`, which fetches, and fast-forwards the parent pair with `FastForwardPair`; a failure of either is Stuck with Halt behind `LandingStopPrefix` naming the error, with `lym loom resume` as the way forward, since the re-entry finds the pull request merged and lands through the already-landed path.
The existing parent-side `Merge` then finds the parent already holding the task's work, marks the task done and the push is a no-op, so the landing ends through the already-landed path.
A merge-in-required error from that `Merge` takes Finalize's existing retry, which re-runs the merge of the parent, the gate and the re-check and calls `Merge` again; it never calls `mergeOnGitHub` a second time, since the GitHub merge is done.
Refusals, each Stuck with Halt behind `LandingStopPrefix` naming the pull request and GitHub's message: `RefusalMethod` names setting landing.yaml's `squash` to the method the repository allows with `lym config landing --set squash=<true|false>` in the prime worktree, or allowing the method in the repository's settings, then `lym loom resume`; `RefusalOther` and a pending refusal past the minute name, for a second person, approving the pull request again or settling the required check or conflict, then `lym loom resume`, and, for the operator alone, `lym loom goto PR-Review` and asking the PR-Review agent to continue, which closes the pull request and lands the task locally through Finalize.
Bound: what reaches the parent after the approval without a second one is the merge of the parent and the finalize fixer's commits, within Finalize's caps and behind the `finalize` gate; GitHub's branch protection can refuse it, which is the halt above; a landing through the already-landed path writes no merge record or conclude commit on the records side.
`TestFinalize_PullRequest`, table-driven over `recordingParentMerger`, `fakePullRequestAPI` and the fake gate the existing Finalize tests use: the resolver runs only after the lock is taken; an unreadable parent mode halts naming the worktree and `lym config beam`; a no closes an open recorded pull request, removes the record and merges locally; a yes with no record halts naming `lym loom goto PR-Review`; a yes with a closed record not closed for continue halts naming reopening it and `lym batten cancel`, and so does one carrying `Continue` without `ClosedForContinue`; `ClosedForContinue` merges locally with no GitHub merge; a merged pull request makes no GitHub merge and lands through the parent-side merge; an open one pushes, merges on GitHub with the pushed head and the squash setting, fast-forwards the parent and marks the task done; a merge-in-required error after the GitHub merge retries the local merge with no second GitHub merge; a pending refusal then a success merges once retried; a pending refusal for over a minute halts; a method refusal halts naming `squash`; another refusal halts naming GitHub's message and `lym loom goto PR-Review`; a transient merge failure is an error; an unresolvable token wrapped in a `*url.Error` halts naming `gh auth login`; a failing `Status` read halts naming the token and repository access; a failing push halts naming `lym fabric push`; a failing fast-forward after the merge halts naming `lym loom resume`; a resolver halt opens with `LandingStopPrefix`.
The doc comments of `Finalize.Call` and `pushParent`, the file comment heading `finalize.go`, and the landingshed doc's sentences that Finalize opens, looks up and closes no pull request and makes no GitHub call, say instead how it lands through the pull request.
`PATTERN.md`'s `PATTERN-parent-lock` line becomes, verbatim:

```
- `PATTERN-parent-lock` — Moving, merging into or landing on a local parent branch, or merging a run's pull request: only in a run's Finalize, under `internal/parentlock`'s lock for that parent.
```

**ImpactSummary:** Finalize gains one step under the lock and one landing branch; a run the resolver says no to, and every hub without a GitHub origin, lands exactly as before, and the existing Finalize tests build with no record and a local origin.

**Commit:** 15: Finalize lands through the run's pull request
