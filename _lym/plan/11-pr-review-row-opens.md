# Card 11 — pr-review-row-opens

**Create:**
- `plan:internal/landingshed#PullRequestDeps` -> `type PullRequestDeps struct`
- `plan:internal/landingshed#fakePullRequestAPI` -> `type fakePullRequestAPI struct`
- `plan:internal/loomcli#TestLandingDeps_PullRequestModeReadsTheParent` -> `func TestLandingDeps_PullRequestModeReadsTheParent(t *testing.T)`

**Edit:**
- `internal/landingshed#Deps`
- `internal/landingshed#PRReview`
- `internal/landingshed#NewPRReview` -> `func NewPRReview(name string, deps Deps) *PRReview`
- `internal/landingshed#PRReview.Call`
- `internal/landingshed#TestPRReview_Call`
- `internal/landingshed/doc.go`
- `internal/shedrecipe#prReviewEntry`
- `internal/shedrecipe#prReviewSettingKey`
- `internal/shedrecipe#prReviewKeyWayForward`
- `internal/shedrecipe#prReviewMissingWayForward`
- `internal/shedrecipe#newPRReviewTestEnv`
- `internal/shedrecipe#prReviewTestConfig`
- `internal/shedrecipe#TestPRReviewEntry`
- `internal/shedrecipe/doc.go`
- `internal/shedbuild#engineMinimalConfig`
- `contracts/recipes/loom-recipe.yaml`
- `contracts/recipes/darn-recipe.yaml`
- `contracts/recipes/pawl-recipe.yaml`
- `internal/loomrecipe#wantPawlRows`
- `internal/loomrecipe#TestNewDarn_Routing`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#landingDeps`
- `internal/testkit/envkit#LandingDeps`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `docs/overview.md`
- `PATTERN.md`
- `pattern/PATTERN-shed-recipe-registry.md`

**Uses:**
- `plan:internal/landingshed#ResolvePullRequest`
- `plan:internal/landingshed#Origin`
- `plan:internal/pullrequest#API`
- `plan:internal/pullrequest#Open`
- `plan:internal/pullrequest#ReadRecord`
- `plan:internal/pullrequest#WriteRecord`
- `plan:internal/pullrequest#RemoveRecord`
- `plan:internal/shedrun#PullRequestRecord`
- `plan:internal/shedrun#PullRequestRecordLock`
- `internal/landingshed#stuckVerdict`
- `internal/landingshed#transientFailure`
- `internal/landingshed#entryErr`
- `internal/loomcli#primeOfHub`
- `internal/loomcli#seedSlug`
- `internal/loomrecipe#TestNewPawl`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomrecipe#TestPawlRouting`
- `internal/loomcli/doc.go`

**Intent:**
D3, D4, D6 and D13: the PR-Review row decides the mode, pushes and opens or finds the pull request, and records it; the stub goes.
Card 13 adds the wait and card 14 the session, so at this card a yes ends Done once the pull request is recorded.

`landingshed.PullRequestDeps`, held as `Deps.PullRequest`, carries the row's seams: `RecordPath` and `RecordLockPath`; `ParentMode func() (mode, worktree string, err error)`, the receiving branch's `beam.yaml` `pull_request` and that worktree's name; `EntryField func() (string, error)`, the landing entry's `pull_request`; `Origin func() (Origin, error)`; `API func(originURL string) (pullrequest.API, error)`; and `PushTask func() error`, the run's blocking both-sides push.

`NewPRReview(name, deps)` keeps the row name and deps. `PRReview.Call`:
1. `entryErr`; a nil `ParentMode`, `EntryField`, `Origin`, `API` or `PushTask`, or an empty `RecordPath`, is an error naming the field (a fail-loud wiring guard).
2. `ParentMode` failing is Stuck with Halt: `the receiving branch's beam.yaml could not be read in worktree <worktree>: <error>; way forward: fix it with lym config beam --set pull_request=<on|off|required> in that worktree, then run lym loom resume`. `EntryField` and `Origin` failing are errors.
3. `ResolvePullRequest(deps.Slug, mode, field, origin)`; a `Halt` is Stuck with Halt carrying it.
4. A no logs the reason at Info; when the record names a pull request that `Status` reports open, it closes it with the comment `lym closed this pull request: <reason>` and removes the record; a recorded pull request GitHub reports merged or closed has its record removed and is left as it is; it returns Done. A failed `Status` or `Close` here is a GitHub failure, handled as below.
5. A yes calls `PushTask`: a transient failure (`transientFailure`) is a returned error, any other is Stuck with Halt naming the push error and `lym fabric push` in the task worktree, then `lym loom resume`.
6. It reads the record. A record marked `ClosedForContinue` belongs to a review the operator already ended, whose pull request lym closed, so a re-entry after a Finalize reroute removes it and opens afresh, so the new commits are reviewed and gated. With no record, `FindOpen(TaskBranch, ParentBranch)` and, with none open, `Create`, so a re-entry, a crash or a reroute through Describe opens no second pull request; with a record it reads `Status` (card 13 handles a closed or merged one; here an open one is reused).
7. On every entry it sets title and body from the change description, read through `summaryparser.Parse(DescriptionPath)` (a parse failure is an error), with `Edit`.
8. It writes the record (number, URL, base, head), keeping any fields already there. `GatedHead` is set to `TaskHead()` after the push only when the record carries none, on the first entry, whose head the pipeline's own gates passed; a re-entry keeps the recorded `GatedHead`, so a commit the agent made before a crash or a resume is pushed to the pull request but stays ungated until card 13's wait gates it, and nothing lym lands rests on it. It returns Done.
A failed GitHub call is judged in this order: `errors.Is(err, githubclient.ErrTokenUnresolvable)` first, since `http.Client` wraps the token error in a `*url.Error` that `githubclient.IsTransient` reads as transient, is Stuck with Halt naming `gh auth login --hostname github.com` run by the operator, then `lym loom resume`; then a transient failure is a returned error through `transientFailure`; any other failure is Stuck with Halt naming the call and the error, with the way forward: make sure the gh user's token (`GH_TOKEN`, `GITHUB_TOKEN`, or `gh auth login --hostname github.com`) is valid and can reach the repository, then `lym loom resume`.
The row takes no parent lock.

`TestPRReview_Call` is rewritten table-driven over a `fakePullRequestAPI` (records every call with its arguments, and holds per method a list of scripted results, one per call, the last repeating) and a temp record path: parent `off` with no field is Done and calls no GitHub; parent `on` with a recorded open pull request and a resolver no (local origin) closes it with the reason and removes the record; an unreadable parent mode halts naming the worktree and the `lym config beam` way forward; a field `on` with a local origin halts with both ways forward; a first yes pushes, finds none, creates one with the description's title and body and writes the record; a re-entry with a record edits title and body and creates none; a yes with none recorded but one open reuses it; a re-entry whose task head is past the recorded `GatedHead` keeps the recorded `GatedHead`; a re-entry whose record is `ClosedForContinue` removes it and creates a new pull request; a no whose recorded pull request is merged removes the record and closes nothing; a non-transient push failure halts naming `lym fabric push`; an unresolvable token, returned wrapped in a `*url.Error` as `http.Client` returns it, halts naming `gh auth login`; a 404 halts naming the token and repository access; a transient GitHub error is an error; a nil seam is an error naming it; a cancelled context is an error.

shedrecipe: `prReviewEntry` takes only the optional `gate_key` (`configRejectUnknown(cfg, "gate_key")`, `prReviewKeyWayForward` becomes `way forward: remove the key; a PRReview row takes only gate_key`), builds its deps with `landingRowDeps`, so its gate key is `pr-review`, and returns `landingshed.NewPRReview(row.Name, deps)`; `prReviewSettingKey` and `prReviewMissingWayForward` go.
`TestPRReviewEntry` covers: an empty config builds a `*landingshed.PRReview`; an unknown key, the retired `pr_review_setting` as the row's input, is refused naming it and the way forward.
The file comments of `internal/landingshed/prreview.go` and `internal/shedrecipe/entries_prreview.go`, which call the row a stub, are rewritten to say what the row and its entry build. `prReviewTestConfig` returns an empty config and `newPRReviewTestEnv` is `newTestEnv`; `engineMinimalConfig`'s `PRReview` entry is empty. The shedrecipe doc's PRReview section says the row takes only `gate_key` and carries no `entry_gate`, since its off reads the parent's anchor, a board field and the origin.

Recipes: the loom and darn `PR-Review` rows drop their `config` and keep `on_done: Finalize` and no `on_stuck`; the pawl recipe gains `- name: PR-Review`, `engine: PRReview`, `on_done: Finalize` after `Beam-Describe`, whose `on_done` becomes `PR-Review`. Every header comment calling the row a stub that always returns Done says instead that it carries no `entry_gate` and no `on_stuck` because its Stuck is always a Halt. `wantPawlRows` and `TestWire_BeamRecipe`'s row list gain the row, and the file comment of `internal/loomrecipe/pawlrecipe_test.go` counts ten rows and names `Beam-Describe`'s next row as `PR-Review`; `TestNewDarn_Routing`'s PR-Review assertion also requires an empty `EntryGate`, and `wantPawlRows`' PR-Review entry carries none, so a row given an `entry_gate` fails both.

loomcli `landingDeps` fills `PullRequest`: record paths from `shedrun.PullRequestRecord(l, shedrun.SelfRunID)` and its lock; `ParentMode` resolves the parent pair through `hubgeom.ResolveParent(l)`, the prime through `primeOfHub` when it names none, and loads `dobbyengine.LoadConfig(fabricengine.ConfigDirs(parent))`, so the receiving branch's anchor decides and the task worktree's own beam.yaml is never read; `EntryField` reads `boardengine.OpenHub(l.HubPath)`'s entry `seedSlug(l.WorktreeName)`, an absent entry reading empty; `Origin` opens the fabric for `OriginURL`, reads `ReadParentTips(parentBranch, false)` for the upstream, and carries `pushSkipped`; `API` is `pullrequest.Open`; `PushTask` is `fabricengine.PushPairAnchored(l, fabricengine.EnvSyncOptions(), fabricengine.StatusPushLockWait)`.
`TestLandingDeps_PullRequestModeReadsTheParent`, in `landingdeps_integration_test.go` (tier1), builds a hub with `hubforge`, sets `pull_request: off` in the prime's beam.yaml and `required` in the task worktree's, and asserts `ParentMode` answers `off`; a second row builds a beam with `hubforge` and a member task under it, sets the beam worktree's `pull_request` to `required` and the prime's to `off`, and asserts that the member's `ParentMode` answers `required` from the beam and the beam's own `ParentMode` answers `off` from the prime, and that the beam's `EntryField` reads its project entry's `pull_request`.
envkit's `LandingDeps` fills every `PullRequest` seam with a stub whose `ParentMode` answers `off` and whose `Origin` is empty, so a test landing through it takes the local path, and the three envkit seam tests gain `//lym:seam internal/landingshed#PullRequestDeps`.

Docs: the landingshed doc's PR-Review paragraph describes the row as above; `docs/overview.md`'s module table line and its PR-Review sentence drop "stub" and say the row lands through a pull request when the resolver says so.
`PATTERN.md`'s `PATTERN-shed-recipe-registry` line becomes, verbatim:

```
- `PATTERN-shed-recipe-registry` — Adding producer or recipe row: one `Constructor` map in `internal/shedrecipe`, bar `Implementer`'s; a row's value looked up by key, a side action a `Command` row, a row off by one config key its `entry_gate`, one whose off reads more its own call. — [background](pattern/PATTERN-shed-recipe-registry.md)
```

and the background file's `## Entry gates` section adds that a row whose off reads more than one config key, such as PR-Review's, which reads the parent's anchor, a board field and the origin, decides it in its own call, never by a recipe rewrite.

**ImpactSummary:** Replaces the stub with the real row and its seams across landingshed, shedrecipe, the three recipes and loomcli's wiring; every caller of `NewPRReview` is the one registry entry and the rewritten test, and the stub's flag readers go with it.

**Commit:** 11: PR-Review opens and records the run's pull request
