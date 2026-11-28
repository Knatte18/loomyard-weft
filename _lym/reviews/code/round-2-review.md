---
verdict: BLOCKING
findings:
  - id: F10
    severity: BLOCKING
    class: design
    location: internal/loomcli/prreview_mail_integration_test.go#prReviewMailDeps
    summary: The F1 fix moved only Mode, Origin and TaskHead to the wired seams; the row still reads envkit's hand-written DirtyPaths (nil), CommitsMissing (nil), TaskHasCommits (true), PauseCovered (false) and RecordChanges (nil channel), standing in for verifytree, fabricengine, the pause record and the record watch over a real pair whose wired seams read them.
    fix: In prReviewMailDeps, after taking MailAgent, Mode and Origin from wired.PullRequest, also assign pr.DirtyPaths = wired.PullRequest.DirtyPaths, pr.CommitsMissing = wired.PullRequest.CommitsMissing, pr.TaskHasCommits = wired.PullRequest.TaskHasCommits, pr.PauseCovered = wired.PullRequest.PauseCovered and pr.RecordChanges = wired.PullRequest.RecordChanges, so every seam the wait reads over git, the pause record or the record file comes from loomcli's wiring over the pair. Where a wired seam needs state the pair lacks (for TaskHasCommits a commit on the pair's HEAD beyond the parent branch), make that state through gitkit or fabric in the test setup rather than keeping the envkit closure. Keep pr.RecordPath and pr.RecordLockPath in step with the wired record paths if RecordChanges watches the wired path. Leave the push counter wrapping whatever PushTask stays, since no row may call it.
  - id: F11
    severity: BLOCKING
    class: decision
    carries: F2
    location: internal/loomcli/prreview_mail_integration_test.go#prReviewMailDeps
    summary: The new test still adds uses of two standing fakes whose inventory verdicts forbid new uses, prkit.API (verdict goes) and creelfake.Delivery through startNoticeOrch (verdict real), and the halt row's unknown-strand refusal rests on the fake delivery's hand-written strand list; the hand-written change description it writes at deps.DescriptionPath is the same kind of agent-output stand-in.
    fix: Leave the code in place and defer this finding to the run's parent. The tree has no recorded GitHub stand-in to drive pullrequest.Client over, no reed-backed creel delivery at tier1 and no recorded change description, so only the parent can rule whether this task builds them or whether the board entry's tier1 test over a real creel daemon accepts the standing fakes. The parent files that ruling as a board note for a new task.
prior:
  - id: F1
    status: resolved
    answer: Mode, Origin and TaskHead now come from loomcli's landingDeps over the pair, beam.yaml's pull_request is set through configengine.Set, the origin URL through git, and the expected head is read with gitkit.RevParse. The seams F1 did not name are raised as F10.
  - id: F2
    status: open
    answer: No added entry in the decision record settles it, so it stays held by the parent and is carried as F11.
  - id: F3
    status: resolved
    answer: The test counts pushes and wants none, compares the whole record, checks the sleep bounds and the parent lock store. The wait-mark check left out is held by the rows staying in TestPRReview_Wait on the same set-and-clear behaviour.
  - id: F4
    status: resolved
    answer: ImplementerSeam.LiveStrands reads the replayed reed's table through reedengine.LiveStrandNames, and the summary's Design decisions say why the batten child seams keep their answers.
  - id: F5
    status: resolved
  - id: F6
    status: resolved
    answer: The round trip in TestPlaceholdCall_HoldsAPastesBufferNameAndHubAndKeepsTheRestOfItsText replays a held stdin under another hub binding, which covers bindCall's Stdin line without writing into a recording.
  - id: F7
    status: resolved
    answer: TestIdleSession_BindsTheBareNameOfAShellTheHostLacks sets LYM_REED_SHELL, which template_posix.yaml reads, and pins the bare name bound.
  - id: F8
    status: resolved
  - id: F9
    status: resolved
---

### [BLOCKING] The mail test still takes the wait's git, pause and watch seams from envkit's canned closures

**Location:** `internal/loomcli/prreview_mail_integration_test.go#prReviewMailDeps`

**Issue:** `prReviewMailDeps` builds its deps from `envkit.LandingDeps`, which sets `CommitsMissing`, `TaskHasCommits`, `DirtyPaths`, `PauseCovered` and `RecordChanges` to fixed answers (`internal/testkit/envkit/envkit.go:262-268`).
The PR-Review row reads every one of them on the paths this test drives: `TaskHasCommits` at `internal/landingshed/prreview.go:111`, `PauseCovered` at :441, `RecordChanges` at :524, `DirtyPaths` at :68 and :700, and `CommitsMissing` at :850 and :912.
The fix for F1 replaced only `Mode`, `Origin` and `TaskHead` with the wired seams, though `wired.PullRequest` (built by `pullRequestDepsGuardedBy` in `internal/loomcli/landingdeps.go`) fills all five over the same real pair.
PATTERN-test-stand-ins forbids a new test hand-writing another module's output when the real module is in reach, and no change may add such a stand-in; this is the same defect F1 named, applied to the seams it did not list.
The test answers the focus question for the rest: the mail text and the head have sources independent of the code (a literal format and `git rev-parse`), the record is written through `pullrequest.WriteRecord`, and the gate command is a scripted gate closure, which the rule allows.

### [BLOCKING] Standing fakes in the new mail test await the parent's ruling

**Location:** `internal/loomcli/prreview_mail_integration_test.go#prReviewMailDeps`

**Issue:** Carried from F2, held by the parent with no added entry settling it.
`prkit.API` answers the pull request status and `startNoticeOrch` runs creel over `creelfake.Delivery`, whose hand-written live-strand list decides the halt row's refusal.
The test also writes the change description by hand at `deps.DescriptionPath`, a file an agent writes and `summaryparser` reads, for which the tree holds no recording.

## Focus results

- Focus 1, the seams of `prReviewMailDeps`: hand-written seams remain, raised as F10; the expected head and mail text have independent sources.
- Focus 2, the three moved rows: every assertion the base loop made for them holds in `TestPRReviewGateFailureMail` no looser, the whole record and the push count included.
  The base asserted no GitHub calls for these rows, since `wantCalls` was nil.
  The wait-mark check is dropped from the moved rows, and the commit carries a `Loosened-Check:` trailer for it.
  The rows that stay in `TestPRReview_Wait` still check the same set-and-clear behaviour, so it is not a removed assertion.
- Focus 3, `tmuxrec`: replay binds `{hub}` in a recorded stdin and matches the paste's text and its one buffer name on any host.
  Other bound values in pasted text stay literal, a choice the summary states.
  Each added behaviour has a test: stdin matching, buffer naming, two names refused, hub-only placeholding with its round trip, and the bare shell name.
  The stdin form of `describe` only renders a failure message.
- Focus 4, `envkit`: `FullEnv`'s live-strand seam and the PR-Review and sampler session seams now run over the replayed reed.
  The batten child seams keep their answers, and the summary says why.
  I judged that a fair reading of the board entry's scope.
- Focus 5, inventory and merge danger: the rows at `loom-landing.md` lines 103 and 200 mark what this diff removed.
  The blast radius names `reedengine`'s two exports and the wiring call, which covers `io.go` and `wiring.go`.
