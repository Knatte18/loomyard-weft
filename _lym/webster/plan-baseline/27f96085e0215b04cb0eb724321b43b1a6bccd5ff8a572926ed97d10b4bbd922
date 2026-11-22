# Card 12 — parent-review-ask-verb

**Create:**
- `plan:internal/loomcli#parentReviewAskVerb` -> `func parentReviewAskVerb(out io.Writer, slug string, deps parentReviewDeps) int`
- `plan:internal/loomcli#TestParentReviewAskVerb` -> `func TestParentReviewAskVerb(t *testing.T)`

**Edit:**
- `internal/loomcli#parentReviewDeps`
- `internal/loomcli#parentReviewState.preRun`
- `internal/loomcli#loomCLI.parentReviewCmd`
- `internal/loomcli#parentAnswerRefusalText`
- `internal/loomcli/doc.go`
- `internal/loomcli#parentReviewFixture`
- `internal/loomcli#TestParentReviewAnswerVerb`

**Extends:**
- `docs/overview.md`

**Uses:**
- `plan:internal/loomcli#prReviewAsker`
- `internal/loomcli#parentReviewWayDecision`
- `internal/loomcli#parentReviewAnswerVerb`
- `internal/loomcli#openParentAsk`
- `internal/landingshed#ReadPRReviewStanding`
- `internal/landingshed#PRReviewAskLineTag`
- `internal/pullrequest#Record`

**Intent:** D4: the PR-Review session's agent sends each re-ask with `lym loom parent-review ask <slug>` (new), and the answer verb names the PR-Review ask's own way forward.

`parentReviewDeps` gains `prRecord func() (pullrequest.Record, bool, error)`, `head func() (string, error)`, `askPR func(rec pullrequest.Record, head string) (burlerengine.ParentAsk, error)` and `now`'s existing clock.
`preRun` fills them for the target worktree: the record at `shedrun.PullRequestRecord(target, shedrun.SelfRunID)`, the head from `fabricengine.Open(target)`'s `HeadSHA`,
and `askPR` as `prReviewAsker(target, rec)` then `Ask(head, now)`.

`parentReviewAskVerb` refuses, writing nothing, each with a trailing way forward (PATTERN-refusal-way-forward):

1. no pull request record: wait until the PR-Review row opens the pull request;
2. a record whose autonomy is not recorded true: the operator ends this review, so follow the operator's ending in the session's prompt;
3. a caller strand (`strandName`) that is not the record's `StrandName`, the empty name included: run it from the PR-Review session's pane;
4. a standing of `PRReviewAwaiting` at the current head: wait for ask N's answer with `lym loom parent-review await <slug>`.

Otherwise it records and posts the ask and answers ok with the ask's number, state, head and deadline.
Bound: only the recorded PR-Review session of an autonomous review can ask, at most one open ask per head, and the verb ends nothing: only `pr-continue` and the row end the review.

`parentReviewCmd` adds the `ask` command (`AudienceRole`, `cobra.MaximumNArgs(1)`) with its Long help,
and the group's and `answer`'s help name the PR-Review ask beside the review and judge asks.
`doc.go` names `parent-review ask` and its callers, and `docs/overview.md`'s loom verb list adds `parent-review ask` beside `answer` and `await`.
A failure of `askPR` after every check passed, a failed record write, is an error envelope naming the cause and running the verb again as its way forward.

`parentAnswerRefusalText`, for a refused ask whose latest ask is tagged `PRReviewAskLineTag` and has ended (past its deadline, not the latest and ended, or already ended),
names waiting for the PR-Review session's next `[pr-review request]` line as its way forward, in place of `parentReviewWayDecision`, since no decision ends a review.
The unnamed-shell caller rule is unchanged, so an operator who turned autonomy off answers the newest PR-Review ask from their own shell.

Tests, tier0. `parentReviewFixture` gains a temp pull request record, a fixed head and a stub `askPR` that records through a real `PRReviewAsker` over the fixture's files with a stub post.

- `TestParentReviewAskVerb`: one row per refusal above, each asserting its message and way forward and that the record holds no new ask;
  a success row records a PR-Review ask with the tag, its number after an earlier Code-Review ask, and the head;
  after that ask reaches its deadline unanswered, a new `ask` is recorded (the timeout criterion);
  an `askPR` stub returning an error answers the error envelope and the record holds no new ask.
- `TestParentReviewAnswerVerb` gains rows: with an earlier Code-Review ask and judge ask in the record, `--satisfied` for the newest PR-Review ask is recorded done;
  the same answer from the unnamed shell (`strandName` empty) is recorded;
  an answer to a PR-Review ask past its deadline is refused naming the next `[pr-review request]` line and not `lym loom decision add`;
  an answer to an earlier PR-Review ask that a newer one replaced, the newer ask ended timeout, is refused the same way;
  the same answer with the newer PR-Review ask still open is refused with the existing "a newer ask, ask N, awaits an answer" text.

**ImpactSummary:** One new verb in the parent-review group and one new branch in the answer refusal's wording; the group's other verbs and their callers are unchanged.
