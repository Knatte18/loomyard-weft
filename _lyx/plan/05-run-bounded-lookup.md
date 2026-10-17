# Card 5 — run-bounded-lookup

**Create:**
- `internal/loomcli#findRunPullRequest`
- `internal/loomcli#TestFindRunPullRequest`

**Edit:**
- `internal/landingshed#FindPullRequest` -> `func FindPullRequest(ctx context.Context, client *github.Client, owner, repo, head, base string, since time.Time) (pr, olderOpen *github.PullRequest, err error)`
- `internal/landingshed#Deps`
- `internal/landingshed#Publish.Call`
- `internal/landingshed#PRGate.Call`
- `internal/landingshed#PRAlign.findPullRequest`
- `internal/landingshed#Finalize.closePullRequest`
- `internal/landingshed#TestFindPullRequest_QueryParametersAndEmptyList`
- `internal/landingshed#TestPublish_ExistingClosedPullRequest`
- `internal/landingshed#TestPublish_OpenPullRequest`
- `internal/landingshed#TestPRGate_Verdicts`
- `internal/landingshed#TestPRAlign`
- `internal/landingshed#TestFinalize_ClosingPullRequestAfterLanding`
- `internal/landingshed/doc.go`
- `internal/loomcli#landingDeps` -> `func landingDeps(l *lyxcwd.Location, recipe string, geom websterengine.Geometry, taskBranch, originURL, parentBranch string, pushSkipped bool, pushBranch func() error, registry modelspec.Registry, runner *shuttleengine.Runner, cfg landingshed.Config, parentName string, languageSkills []string, verifyWaitMark func(label string, start time.Time) error, stopConflictSession func() (string, error), verify verifySource, lock landingshed.ParentLockDeps, runStart time.Time) landingshed.Deps`
- `internal/loomcli#loomCLI.wireLanding`
- `internal/loomcli#approveDeps`
- `internal/loomcli#approveVerb`
- `internal/loomcli#loomCLI.approveCmd`
- `internal/loomcli#rejectDeps`
- `internal/loomcli#rejectVerb`
- `internal/loomcli#loomCLI.rejectCmd`
- `internal/loomcli#approveFixture`
- `internal/loomcli#rejectFixture`
- `internal/loomcli#TestApproveVerb_Refusals`
- `internal/loomcli#TestApproveVerb_Success`
- `internal/loomcli#TestRejectVerb_Refusals`
- `internal/loomcli#TestRejectVerb_Success`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/loomcli#markDoneFixtureDir`
- `internal/loomcli#TestLandingDeps_VerifyCommandReadsPlanAtCallTime`
- `internal/loomcli#TestWire_Real_PublishRejectedPushNamesRemoteTip`
- `internal/loomcli#TestLandingDeps_ConfigNoticeSeamsOverRealHub`
- `internal/loomcli#TestWireLanding_TellsLanguageSkills`

**Uses:**
- `internal/loomcli#seedRunStart`
- `internal/landingshed#newTestDeps`
- `internal/landingshed#newPRGateFixture`
- `internal/landingshed#publishGitHubServer`
- `internal/landingshed#installCloseServer`

**Intent:** D1: a run's pull request is one created at or after the run started, and every landing step resolves it the same way.

`FindPullRequest` keeps its one list query (state all, head `owner:head`, base, sorted by creation date descending) and gains `since`, the run's start.
It returns as `pr` the first listed pull request, whatever its state, whose `GetCreatedAt().Time` is not before `since`, so one created exactly at `since` is the run's; an older one reads as none.
It returns as `olderOpen` the first listed pull request created before `since` whose state is `open`, the newest such, from the same response, for Publish's stop in the next card; nil when there is none.
A zero `since` filters nothing: `pr` is the newest pull request and `olderOpen` is nil, today's behaviour.
An empty list returns two nils and a nil error; a GitHub error is returned as-is with two nils.
Its doc comment gains the run-start bound, the older open pull request it also reports, and that a zero bound keeps the unbounded lookup.
`internal/landingshed/doc.go`'s Publish paragraph (`doc.go:6`) gains one sentence: every landing row's pull-request lookup takes the newest pull request created at or after the run's start, `Deps.RunStart`, whatever its state, and a zero start keeps the unbounded lookup.

`Deps` gains `RunStart time.Time`: the run's start, the `started_at` stamp of the pair's seed, told by the caller and derived by nothing here (`PATTERN-told-geometry`); its comment says every pull-request lookup in this package is bounded by it and that zero means unbounded.
`Publish.Call`, `PRGate.Call`, `PRAlign.findPullRequest` and `Finalize.closePullRequest` each pass `deps.RunStart` and take `pr`; only Publish reads `olderOpen`, in card 6, so this card's Publish discards it.
With the bound, a closed or merged pull request from before the run reads as none: Publish creates a new pull request and PR-Gate awaits with its existing no-pull-request reason instead of ending Done on an old merge; PR-Align aligns nothing and Finalize closes nothing for an older open one.

`internal/loomcli` reads the bound once per wiring with card 4's `seedRunStart`:
`loomCLI.wireLanding` calls it on `c.location` before `landingDeps` and passes the time as `landingDeps`' new last argument, which fills `Deps.RunStart`.
`seedRunStart`'s error covers a seed that does not decode or fails validation as well as one that cannot be read, so it is a correctness halt and not an exempt raw I/O failure: `wireLanding` returns it as `%w; way forward: rm <seed>, then lyx shed seed self --recipe loom --driver go --param parent=<parent>`, `<seed>` being `shedrun.SeedFile(c.location, shedrun.SelfRunID)` and `<parent>` the parent branch `landingBranches` returned, the re-seed `resolveRunID`'s missing-seed remedy names.
The seed read at `internal/loomcli/sharedbootstrap.go:100` always reads the self seed and refuses an undecodable one first on the start path.
`loomCLI.resolveRecipe` (`internal/loomcli/arm.go:127-133`) refuses it first only when the addressed run is the self run, because it reads the seed of `c.runID` while `seedRunStart` always reads the self seed; for `lyx loom run <run-id>` or `lyx loom step <run-id>` naming another run with a healthy seed, an undecodable self seed reaches `wireLanding`'s branch and fails the landing wiring.
The approve and reject verbs gain a seam `runStart func() (time.Time, error)` on `approveDeps` and `rejectDeps`, filled in `approveCmd` and `rejectCmd` with a closure returning `seedRunStart(location)`, and their `findPR` field gains a trailing `since time.Time` parameter.
The two inline `findPR` closures are replaced by one named function, `findRunPullRequest`, in `internal/loomcli/runstart.go`: it builds the client through `landingshed.NewGitHubClient`, passes `since` to `FindPullRequest` and returns its `pr`, and both `approveCmd` and `rejectCmd` set `findPR: findRunPullRequest`, so neither verb has a closure of its own that could drop the bound.
`runstart.go`'s file header gains the run-bounded pull-request lookup the approve and reject verbs use, beside the seed's start read.
`approveCmd`'s and `rejectCmd`'s `Long` help, which lists what each verb refuses, gains that it refuses when the run's seed cannot be read; `TestApproveCmd_HelpNamesAwaitingOrBlocked`'s phrase "awaiting or blocked at PR-Gate" stays.
`approveVerb` and `rejectVerb` call `runStart` right after the origin URL parses and before `findPR`, and refuse on its error with `loom: approve: read the run's start: <err>; way forward: transient, re-run lyx loom approve`, and the same with `reject` in `rejectVerb`, writing no record;
the approve message matches `approveVerb`'s sibling read refusals, and the reject message carries the way-forward clause though `rejectVerb`'s sibling read refusals carry none; the zero bound passes through as `since` unchanged.
`loomCLI.resolveRecipe` (`internal/loomcli/arm.go:127-133`) already refuses an unreadable or undecodable self seed in every approve and reject invocation's persistent pre-run (`internal/loomcli/cli.go:230-252`, through `armAt` at `arm.go:184`), before the verb body runs.
So the verbs' own branch is a propagated read failure, which `contracts/specs/refusal-spec.md` exempts as transient, and D1's "the approve and reject verbs refuse with it" is carried by that upstream refusal; the `SeedUnreadable` rows below reach the verbs' branch through the `runStart` seam.

What the bound lets through: it only narrows what a lookup returns, so no step acts on a pull request it did not find before; a zero bound, from a seed with no usable stamp, keeps today's lookup and is logged by `seedRunStart`.

Tests, as rows of the existing tables (`PATTERN-test-economy`); the GitHub fakes already serve any list body, so each row only adds `created_at` to its listed pull requests and sets `deps.RunStart` to `2026-06-01T00:00:00Z`, calling the earlier stamps "before" and later ones "after":
- `TestFindPullRequest_QueryParametersAndEmptyList` becomes a table over list bodies and `since`, keeping the query-parameter assertions on every row: the empty list; a zero `since` over an older closed pull request returns it as `pr`; a `since` with only an older closed one returns two nils; a newer closed one above an older open one returns the newer as `pr` and the older as `olderOpen`; an older closed one listed above an older open one returns a nil `pr` and the open one as `olderOpen`; two older open ones return the first listed as `olderOpen`; one created exactly at `since` is `pr`.
- `TestPublish_ExistingClosedPullRequest` gains `runStart` and `wantCreates` fields, with `wantCreates` 0 on its existing rows: an older closed unmerged pull request makes Publish create one and end Done; an older merged one makes it create one and end Done; a closed unmerged one created after the run started still ends Stuck with "the pull request was closed without being merged".
- `TestPublish_OpenPullRequest` gains a `runStart` and a `createdAt` field: an open pull request created after the run started with a differing description is edited once and none is created.
- `TestPRGate_Verdicts` gains: an older merged pull request awaits naming no pull request (`reasonHas` "no pull request exists"); an open pull request created after the run started with a matching approval is Done.
- `TestPRAlign` gains: an older open pull request spawns nothing and records nothing; an open pull request created after the run started is aligned and its approve applied.
- `TestFinalize_ClosingPullRequestAfterLanding` gains: an older open pull request is looked up and left alone (`lookupOnly`); an open one created after the run started is commented on and closed (`lookupCommentClose`).
- `TestApproveVerb_Refusals` and `TestRejectVerb_Refusals` gain a `SeedUnreadable` row whose `runStart` returns an error, wanting the message to carry "read the run's start" and the way forward "transient, re-run lyx loom approve", and `reject` in the reject row; `approveFixture` and `rejectFixture` fill `runStart` with a fixed time, and their `findPR` fakes take the new parameter, as do the rows that replace `findPR`.
- `TestApproveVerb_Success` and `TestRejectVerb_Success` assert the `since` their `findPR` fake received equals the fixture's `runStart` time.
- `TestFindRunPullRequest`, new and serial because it swaps the package-level `landingshed.NewGitHubClient` (named in its comment), points the client at a local `httptest` server listing a pull request created before `since` and asserts nil, then one created after `since` and asserts it is returned, so a `findRunPullRequest` that drops `since` fails; the `runStart` closures are a one-line call of `seedRunStart`, which `TestSeedRunStart` covers.
- `TestLandingDeps_EveryFieldPopulated` passes a non-zero `runStart`, so its reflection walk keeps every field populated; `markDoneFixtureDir`, `TestLandingDeps_VerifyCommandReadsPlanAtCallTime`, `TestWire_Real_PublishRejectedPushNamesRemoteTip` and `TestLandingDeps_ConfigNoticeSeamsOverRealHub` pass `time.Time{}` to the re-signed `landingDeps`.
- `TestWireLanding_TellsLanguageSkills` goes on after its existing assertion: it writes `shedrun.SeedFile(location, shedrun.SelfRunID)` holding a seed whose `started_at` is `2026-10-01T08:00:00Z`, re-runs `wireLanding` and asserts `c.env.Landing.RunStart` equals that time;
it then overwrites that seed file with `{not json`, calls `c.wireLanding()` again and asserts a non-nil error whose text carries the seed file's path and the way forward `then lyx shed seed self --recipe loom --driver go --param parent=`, pinning that a seed that exists but does not decode fails the landing wiring.

**ImpactSummary:** `FindPullRequest`'s callers, the four landing rows and the approve and reject closures that `findRunPullRequest` replaces, all change in this card; `landingDeps`' one production caller `wireLanding` and its five test callers move with its re-sign, and the card adds two failure branches: the approve and reject verbs' refusal on their `runStart` read, and `wireLanding`'s return of `seedRunStart`'s error for a self seed that cannot be read or decoded.

**Commit:** 5: Bound every landing pull-request lookup by the run's start
