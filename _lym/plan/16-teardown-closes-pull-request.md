# Card 16 — teardown-closes-pull-request

**Create:**
- `plan:internal/pairteardown#closeOpenPullRequest` -> `func closeOpenPullRequest(ctx context.Context, slug string, gone bool, rec pullrequest.Record, found bool, api func(originURL string) (pullrequest.API, error), originURL string)`
- `plan:internal/pairteardown#Teardown.closePairPullRequest` -> `func (t *Teardown) closePairPullRequest(slug string)`
- `plan:internal/pairteardown#TestCloseOpenPullRequest` -> `func TestCloseOpenPullRequest(t *testing.T)`
- `plan:cmd/lym#TestPullRequestGuard_OnlyLandingAndTeardown` -> `func TestPullRequestGuard_OnlyLandingAndTeardown(t *testing.T)`

**Edit:**
- `internal/pairteardown#Teardown`
- `internal/pairteardown#New`
- `internal/pairteardown#Teardown.RemovePair`
- `internal/pairteardown#Teardown.EndForCancel`
- `internal/pairteardown#TestEndForCancel`
- `internal/pairteardown#fakeSubstrates`
- `internal/pairteardown#fakeSubstrates.teardown`
- `internal/pairteardown#TestTeardown_CallSequence`
- `internal/pairteardown/doc.go`
- `PATTERN.md`

**Uses:**
- `plan:internal/pullrequest#API`
- `plan:internal/pullrequest#Open`
- `plan:internal/pullrequest#ReadRecord`
- `plan:internal/shedrun#PullRequestRecord`
- `plan:internal/shedrun#PullRequestRecordLock`
- `internal/pairteardown#Teardown.taskLocation`
- `internal/testkit/logcapture#CaptureVerbose`
- `cmd/lym#TestGHGuard_NoShellOutOutsideGithubclient`

**Intent:**
D12: a pair torn down unlanded leaves no open pull request that looks like pending work.
`Teardown` gains `closePR func(slug string)`, which `New` wires to `closePairPullRequest`, and `RemovePair` calls it first, after the snapshot and before the archive and the removal, so the record under the run's scratch directory is still there; every removing route goes through `RemovePair` (`lym batten cancel --remove`, `lym cleanup --apply`, `lym fabric remove` and batten's Worktree-Teardown), so each closes it.
A plain `lym batten cancel`, which keeps the pair, never reaches `RemovePair`, so `EndForCancel` calls `closePR` too, after the strand stop and before the session end, with the task worktree present; the cancel ends the run for good, and several halts name it as their way forward, so its pull request is closed there as well.
`closePairPullRequest` reads whether the task worktree is gone through `t.gone`, and when present resolves the task location, reads the record at `shedrun.PullRequestRecord(task, shedrun.SelfRunID)` and the pair's origin URL through its fabric, then calls `closeOpenPullRequest` with `pullrequest.Open` as the API factory.
`closeOpenPullRequest`: a gone task worktree logs a Warn naming the slug and that a pull request whose head is its task branch may stay open, and calls nothing; no record calls nothing; a record whose pull request `Status` reads open is closed with one comment, `lym closed this pull request: the run's pair was torn down before it landed`; merged or closed is left as it is, so a landed run's merged pull request stays; any failure (an unreadable record, the client, the status read, the close) logs a Warn naming the pull request's URL where known and never stops the teardown.
The remote task branch is handled as today.
`fakeSubstrates` and its `teardown` give the fake teardown a `closePR` that records its calls; `TestTeardown_CallSequence` asserts it runs once before the remove and also on a local removal; `TestEndForCancel` asserts a cancel keeping the pair calls it once.
`TestCloseOpenPullRequest` (new, in `teardown_test.go`, which is tier0; it stays serial, since it captures the process-global logger through `logcapture.CaptureVerbose`) is table-driven over a fake `pullrequest.API`: an open pull request is closed with the comment; a merged one and a closed one are left; no record calls nothing; a gone worktree warns naming the slug and calls nothing; a failing close and a failing status each warn naming the URL and return; an unreadable record and a failing API factory (an origin URL it cannot use) each warn and return.
`TestPullRequestGuard_OnlyLandingAndTeardown`, a new top-level test beside `TestGHGuard_NoShellOutOutsideGithubclient` in `cmd/lym/ghguard_test.go` (tier0, package `main`), enforces the new clause of `PATTERN-github-auth` that the entry marks `(test)`: it walks every non-test `.go` file of the module and fails on a file outside `internal/pullrequest` that calls one of go-github's mutating pull request calls, `.PullRequests.Create(`, `.PullRequests.Edit(` or `.PullRequests.Merge(` (the read-only `.PullRequests.List(` calls of `internal/loomcli/runstart.go` and `internal/fabriccli/openprheads.go` stay allowed, since they open, merge and close nothing), and on a file outside `internal/landingshed`, `internal/pairteardown` and `internal/loomcli`, which wires their seams and the `pr-continue` verb, that imports `internal/pullrequest`, naming the file and the way forward of each.
The pairteardown doc gains a `# Pull request` paragraph.
`PATTERN.md`'s `PATTERN-github-auth` line becomes, verbatim:

```
- `PATTERN-github-auth` — Calling GitHub: auth only via internal/githubclient; no other production package runs gh; only lym selfreport create files an issue; only PR-Review, Finalize and pair teardown open, merge or close a pull request. (test)
```

**ImpactSummary:** One best-effort step at the head of `RemovePair` and in `EndForCancel`, plus one guard test; it never fails a teardown, and a pair with no record makes no GitHub call.

**Commit:** 16: pair teardown closes the run's open pull request
