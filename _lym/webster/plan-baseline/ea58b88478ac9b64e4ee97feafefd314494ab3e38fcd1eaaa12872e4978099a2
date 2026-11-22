# Card 10 — pr-review-session-autonomy

**Create:**
- `plan:internal/loomcli#prReviewAsker` -> `func prReviewAsker(l *lymcwd.Location, rec pullrequest.Record) (landingshed.PRReviewAsker, error)`
- `contracts/stencils/landing/landing-block-pr-review-operator.md`
- `contracts/stencils/landing/landing-block-pr-review-autonomous.md`
- `plan:contracts/stencils#LandingBlockPRReviewOperator` -> `var LandingBlockPRReviewOperator []byte`
- `plan:contracts/stencils#LandingBlockPRReviewAutonomous` -> `var LandingBlockPRReviewAutonomous []byte`

**Edit:**
- `internal/landingshed#PullRequestDeps`
- `internal/landingshed#optionalPullRequestSeams`
- `internal/landingshed#PRReview.ensureSession`
- `internal/landingshed#PRReview.sessionSpec`
- `internal/landingshed#RenderPRReviewPrompt` -> `func RenderPRReviewPrompt(deps Deps, prURL, engine string, autonomous bool) (string, error)`
- `internal/landingshed#PRReview.askParentReview`
- `internal/landingshed/doc.go`
- `contracts/stencils/landing/landing-template-pr-review.md`
- `contracts/stencils#LandingTemplatePRReview`
- `contracts/stencils#entries`
- `internal/shuttleparity#buildPRReview`
- `internal/loomcli#pullRequestDepsGuardedBy`
- `internal/loomcli#intentionallyNil`
- `internal/battencli#intentionallyNil`
- `internal/testkit/envkit#LandingDeps`
- `internal/landingshed#TestPRReview_Session`
- `internal/landingshed#prReviewTestDeps`

**Uses:**
- `internal/landingshed#PRReviewAsker`
- `internal/landingshed#PRReviewAsker.Ask`
- `internal/landingshed#ReadPRReviewStanding`
- `internal/landingshed#PRReviewStanding`
- `internal/loomcli#runAutonomy`
- `internal/loomengine#Config`
- `internal/pullrequest#Record`
- `internal/landingshed#fakePRSessions`
- `internal/landingshed#newPRReviewFixture`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`

**Intent:** D4: PR-Review records the run's autonomy once per review and, under it, runs a non-interactive session the parent reviews through recorded asks.

Seams. `PullRequestDeps` gains three fields, each named in `optionalPullRequestSeams`:

- `Autonomy func() bool` reads the run's autonomy at call time; nil reads false.
- `AskPRReview func(head, prURL string) (burlerengine.ParentAsk, error)` records and posts a PR-Review ask at `head`.
- `PRReviewStanding func(head string) (PRReviewStanding, error)` reads the newest PR-Review ask against `head`.
- The last two are needed only under a recorded autonomy of true; there a nil one is a fail-loud wiring error naming the field, as `checkWiring`'s errors do.

The row:

- `ensureSession`, on the path that starts or resumes a session and before it builds the spec, sets `rec.Autonomy` from `Autonomy()` when the record holds none,
  and never otherwise; `sessionSpec` then reads the value just set.
  The value is written in the same `updateRecord` that writes the session fields, so a failed start records nothing and the next start reads the layers again.
  So every later session start of the same review, after a pause or a judging stop, builds its spec from the recorded value whatever the layers say now.
- `sessionSpec` sets `Interactive` to the negation of the recorded value and passes it to `RenderPRReviewPrompt`.
- `RenderPRReviewPrompt` fills a new marker, `review_mode`, with one of two stencil blocks it reads through `stencilstore.ReadForEngine` and fills with the slug,
  so the mode text stays in the stencil registry (PATTERN-stencil-ownership).
  - `landing-block-pr-review-autonomous.md` says the parent orch reviews in the operator's place and no operator is in the pane.
    After each set of fixes the agent commits, asks with `lym loom parent-review ask <slug>`, and waits with `lym loom parent-review await <slug>`.
    On findings it runs `lym loom pr-findings <file>`; it runs `lym loom pr-continue` only once the parent's answer is satisfied at its current head.
    When `await` reports an ask ended timeout, no-parent or not-sent, or no ask stands at its head, it asks again with `lym loom parent-review ask`;
    after a no-parent end it first ends its turn and asks again only on the next mail, so a parent that is down raises hold notices instead of a loop of asks.
  - `landing-block-pr-review-operator.md` holds today's "Your parent's review" and "The operator's ending" sections, moved unchanged.
  - The stencil puts `{{.review_mode}}` where those two sections stood, and its other sections lose their address to an operator in the pane:
    the title, the opening lines that the operator discusses the pull request in this pane, "Answer the operator's questions" and "wait for the operator's next message"
    move into the operator block, and the shared lines speak of the reviewer, who is the operator or the parent as the block says.
    The leading comment's marker list names `review_mode`.
  - `stencils.go` embeds the two blocks, `entries` registers them, and `LandingTemplatePRReview`'s comment no longer calls the session the operator's.
- `askParentReview` under a recorded autonomy of true asks once per record until a send-back clears `ParentReviewAsked`, as today,
  through `AskPRReview(head, rec.URL)` with `head` from `TaskHead()`, whatever `ParentReviewWanted` reads.
  An ask that returns an error is logged and leaves `ParentReviewAsked` unset, so the next entry asks again.
  An ask recorded in any state, a no-parent one included, sets `ParentReviewAsked`, and the session asks again as its block says.
  With the recorded value false it runs as today.

Wiring:

- `prReviewAsker(l, rec)` builds the asker for the run at `l`: `Files` from `burlerengine.ParentReviewFilesOf(l.AnchorPath())`,
  `Slug`, `URL` from `rec.URL`, branches from `rec.Head` and `rec.Base`,
  `Wait` from loom's `pr_review_parent_timeout_min` loaded over `l`'s layers,
  `Resolve` as `orchcli.ParentStrand(l)` and `Post` as `orchcli.PostAsProgram(l, strand, loomProgram, line, creelengine.PostTags{Run: l.WorktreeName})`,
  as `driftDeps` wires the judge seat.
- `pullRequestDepsGuardedBy` fills `Autonomy` with `runAutonomy(l)`, `AskPRReview` with the record read at the call and `prReviewAsker(...).Ask(head, time.Now())`,
  and `PRReviewStanding` with `landingshed.ReadPRReviewStanding(burlerengine.ParentReviewFilesOf(l.AnchorPath()), head, time.Now())`.
- `envkit.LandingDeps` fills the three seams with stubs, so `TestFullEnv_FillsEverySeam` stays green.
- `loomcli`'s and `battencli`'s `intentionallyNil` gain `Landing.PullRequest.Autonomy`, `.AskPRReview` and `.PRReviewStanding`
  with the reason their siblings carry (Env.Landing assembled in run.go; batten lands no branch).
- `buildPRReview` passes `false`.
- `doc.go` states the recorded autonomy and the session's interactivity.

Tests, tier0. `prReviewTestDeps` returns a bare `Deps`, so it fills the three seams with defaults:
`Autonomy` reads false, `AskPRReview` returns an ask numbered 1, and `PRReviewStanding` answers `ask-again`.
Each new row sets `deps.Landing.PullRequest.Autonomy`, `AskPRReview` and `PRReviewStanding` itself, the ask stub appending to a slice the row reads.
`TestPRReview_Session` gains rows:

- autonomy true on a record holding none: the started spec has `Interactive` false, the record holds autonomy true, and one PR-Review ask went out;
- autonomy false: `Interactive` true, record false, no ask unless `ParentReviewWanted`;
- a record already holding autonomy true with `Autonomy` now reading false, after a pause stopped the session, and again after a judging stop:
  each restarted spec is still non-interactive (the toggle criterion);
- a record holding autonomy true with `ParentReviewAsked` asks none;
- autonomy true with `AskPRReview` returning an error: the record keeps `ParentReviewAsked` unset and the next entry asks again.

`TestLandingDeps_EveryFieldPopulated` walks `PullRequestDeps` and passes with the three fields `pullRequestDepsGuardedBy` fills.

**ImpactSummary:** PR-Review's session start and first ask read a recorded value; three optional seams join PullRequestDeps with every wiring and seam waiver list updated, and `RenderPRReviewPrompt`'s two callers are both on this card.
