# Card 9 — pr-review-ask-helpers

**Create:**
- `plan:internal/landingshed#PRReviewAskLineTag` -> `const PRReviewAskLineTag = "[pr-review request]"`
- `plan:internal/landingshed#PRReviewAskSeed` -> `const PRReviewAskSeed = "pr-review"`
- `plan:internal/landingshed#PRReviewAsker` -> `type PRReviewAsker struct`
- `plan:internal/landingshed#PRReviewAsker.Ask` -> `func (a PRReviewAsker) Ask(head string, now time.Time) (burlerengine.ParentAsk, error)`
- `plan:internal/landingshed#PRReviewAskLine` -> `func PRReviewAskLine(slug, prURL, taskBranch, parentBranch string, ask burlerengine.ParentAsk) string`
- `plan:internal/landingshed#PRReviewStanding` -> `type PRReviewStanding string`
- `plan:internal/landingshed#PRReviewSatisfied` -> `const PRReviewSatisfied PRReviewStanding = "satisfied"`
- `plan:internal/landingshed#PRReviewAwaiting` -> `const PRReviewAwaiting PRReviewStanding = "awaiting"`
- `plan:internal/landingshed#PRReviewAskAgain` -> `const PRReviewAskAgain PRReviewStanding = "ask-again"`
- `plan:internal/landingshed#ReadPRReviewStanding` -> `func ReadPRReviewStanding(files burlerengine.ParentReviewFiles, head string, now time.Time) (PRReviewStanding, error)`
- `plan:internal/landingshed#TestPRReviewAsker_Ask` -> `func TestPRReviewAsker_Ask(t *testing.T)`
- `plan:internal/landingshed#TestReadPRReviewStanding` -> `func TestReadPRReviewStanding(t *testing.T)`

**Delete:**
- `internal/landingshed#parentReviewRequestPrefix`

**Edit:**
- `internal/landingshed#PRReview.parentReviewRequest`
- `internal/landingshed#TestPRReview_Session`

**Uses:**
- `internal/burlerengine#ParentAsk`
- `internal/burlerengine#ParentAskRecord`
- `internal/burlerengine#EndParentAsk`
- `internal/driftengine#JudgeParentSeat`

**Intent:** D4: a PR-Review ask lives in burlerengine's run-wide ask record, and the row, `lym loom parent-review ask`, `pr-continue` and the row's `judge` share one asker and one reading of it.
They live in a new `landingshed` file, which imports `burlerengine` (no cycle: burlerengine imports no landing package).

- `PRReviewAskLineTag` replaces the unexported `parentReviewRequestPrefix`, with the same text, so the orch's role and the record's tag name one line.
  `parentReviewRequest` uses it.
- `PRReviewAsker` mirrors `driftengine.JudgeParentSeat`: fields `Files burlerengine.ParentReviewFiles`, `Slug`, `URL`, `TaskBranch`, `ParentBranch`,
  `Wait time.Duration` (`pr_review_parent_timeout_min`), `Resolve func() (strand string, recorded bool, err error)` and `Post func(strand, line string) error`.
- `Ask` records an ask with `Seed` `PRReviewAskSeed`, `Tag` `PRReviewAskLineTag`, `Digest` `head` and `AskedAt` `now`, its number the run's next.
  A parent that resolves records it pending-post with `Strand` and `Deadline` `now + Wait`, posts `PRReviewAskLine`, and marks it posted;
  a failed post ends it no-parent with the error as its reason.
  A resolve error or a parent with no strand records it no-parent and posts nothing.
  It returns the ask as it then stands.
- `PRReviewAskLine` opens with the tag, then `<slug> ask <n>:`, names the pull request, branches, head and the deadline as a wall-clock time,
  and gives the answer as `lym loom parent-review answer <slug> --ask <n> --satisfied` or `... --findings-file <path>`.
- `ReadPRReviewStanding` reads the record's newest ask tagged `PRReviewAskLineTag` (`ParentAskRecord.LatestTagged`), so a Code-Review, Discussion-Review or judge ask of the same run is never read as one:
  - `PRReviewSatisfied` when it was sent at `head` and is done with a report holding no findings;
  - `PRReviewAwaiting` when it was sent at `head` and is pending-post or open before its deadline;
  - `PRReviewAskAgain` otherwise: no PR-Review ask, one sent at another head, one with findings, one ended timeout, no-parent or cleared, or one open past its deadline.

Tests, tier0, over a temp `burlerengine.ParentReviewFilesOf` dir:

- `TestPRReviewAsker_Ask`: a resolving parent gets one post whose line holds the tag, `ask 1`, the head and `lym loom parent-review answer the-slug --ask 1 --satisfied`,
  and the record's ask carries the tag, seed, digest, strand, open state and `now + Wait`;
  an earlier Code-Review ask makes the new ask's number 2;
  a parent with no strand records no-parent and posts nothing; a failing post records no-parent with its reason.
- `TestReadPRReviewStanding`: one row per standing above, including a satisfied ask at an older head (`ask-again`),
  an open ask past its deadline (`ask-again`), and a newer Code-Review ask after a satisfied PR-Review ask at the head (`satisfied`).
- `TestPRReview_Session`'s request assertion names `PRReviewAskLineTag` in place of the deleted constant.

**ImpactSummary:** New landingshed surface with no caller until card 10; the deleted constant's two references, the request builder and one session-test assertion, move to the exported tag in this card.
