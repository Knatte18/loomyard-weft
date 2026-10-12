# Card 7 — loom-checklist

**Create:**
- `plan:internal/loomrecipe#RowNames` -> `func RowNames(recipe string) ([]string, error)`
- `plan:internal/loomrecipe#TestRowNames_FollowEachRecipesRowOrder` -> `func TestRowNames_FollowEachRecipesRowOrder(t *testing.T)`
- `plan:internal/loomcli#loomCLI.checklistCmd` -> `func (c *loomCLI) checklistCmd() *cobra.Command`
- `plan:internal/loomcli#renderChecklist` -> `func renderChecklist(rows []string, rowsErr error, poll statuspane.StatusPoll, startedAt string, now time.Time) statuspane.Frame`
- `internal/loomcli/checklist_test.go`

**Delete:**
- `internal/loomcli#applyStatusStrandSurface`
- `internal/loomcli#removeStatusStrands`
- `internal/loomcli#TestApplyStatusStrandSurface`
- `internal/loomcli#TestRemoveStatusStrands`

**Edit:**
- `internal/loomcli#Command`
- `internal/loomcli#loomCLI.resolvePersistentPreRun`
- `internal/loomcli#statusStrandAddSpec`
- `internal/loomcli#statusStrandCmd`
- `internal/loomcli#loomCLI.startCmd`
- `internal/loomcli#mustUseLLMDriverArm`
- `internal/loomcli#TestStatusStrandCmd`
- `internal/loomcli#TestStatusStrandAddSpec`
- `internal/loomcli#TestSmokeStatusStrandAcrossDriverSeeds`
- `internal/loomcli/doc.go`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/statuspane#FormatMinutes`

**Intent:**
Build D7: the pair's `loom-status` strand becomes the gate checklist, a top content strand at a quarter share, on both child drivers.

`RowNames(recipe)` returns the row names of the loom recipe (`shedrun.RecipeLoom`) or the darn recipe (`shedrun.RecipeDarn`) in file order, parsed through `shedbuild.Parse`, the sole recipe parser (`PATTERN-sole-parsers`), and refuses any other name with an error naming it.

`lyx loom checklist` is a hidden pane verb, audience internal, with a `Short` and JSON errors (`PATTERN-cli-cobra`).
`resolvePersistentPreRun` skips it, as it skips the review groups, because arming refuses a missing seed and a pane must outlive one.
Its body resolves the worktree from the command's cwd (`lyxcwd.CwdFrom`, `lyxcwd.Resolve`), reports a resolution failure on the JSON envelope, and builds the worktree's reed engine (`reedengine.LoadConfig`, `hubgeom.ReedGeometry`); then it runs a `statuspane.Pane` on stdout with `statuspane.TerminalSize(os.Stdout)`, `statuspane.ReportFromEnv(os.Getenv, engine.SetContentRows)`, `time.Sleep`, and a poll that reads the self run's seed through `shedrun.ReadSeed` until one read succeeds and then the seed's recipe rows through `RowNames` once, handing `renderChecklist` the seed's or `RowNames`' error as `rowsErr` while either fails (an absent seed is such a failure), and the status file `shedrun.StatusFile(location, shedrun.SelfRunID)` through a `statuspane.StatusCache`, then calls `renderChecklist`.
Bound: it reads files only; its one write is the content-rows report, which card 3's checks confine to its own strand.

`renderChecklist` (pure, in `checklist.go`) returns the frame:
- The first line is the state line: `shedverbs.RenderStatusLine("loom", status)`, or `shedverbs.UnavailableLine("loom")` while the status is not found or unreadable.
- With `rowsErr` set, the only other line is `recipe unreadable`.
- Otherwise one line per recipe row in recipe order: a box, `[x]` done, `[>]` running, `[!]` failed, `[-]` halted, `[ ]` not reached; the row name; the time spent in it summed over its visits, rendered through `statuspane.FormatMinutes`; and `x<N>` when it was visited more than once.
- Visits follow D7: a visit lasts from the previous history entry's end (the seed's `started_at` for the first entry; with none, the first entry's `at`) to its own end, its `last_at` when set and else its `at`; an entry's `repeats` count as further visits and its end is its `last_at`; a `goto` entry is no visit and adds no time, and its `at` starts the next visit; a call that recorded no entry adds its time to the next recorded visit, since the start does not move.
- The current producer's row is running while the state is running, failed while it is failed and halted while it is paused, blocked or awaiting, and its current visit runs from the last entry's end to `now` in every one of them; on a done run no visit is open.
  Every other row with a visit is done, and a row with none is not reached.
- History entries naming a row the recipe no longer has follow the recipe rows, each marked `(not in recipe)`.
- Its `Fit(rows)` keeps the state line and the running row (else the last visited row), drops the rows furthest from it first, and ends with `… <N> more rows not shown`.

The bootstrap runs the checklist in the existing strand: `statusStrandCmd` composes `<exe> loom checklist` through the shell seam; `statusStrandAddSpec` declares `Anchor: render.AnchorTop` and `Size{Kind: render.SizeContent, ShareNum: 1, ShareDen: 4, MinRows: 3}` from named constants; the strand keeps its name, its sidecar and its keep-or-replace logic.
`startCmd` calls `ensureStatusStrand` on both drivers, so `applyStatusStrandSurface` and `removeStatusStrands` go; the comments of `startCmd`, `mustUseLLMDriverArm` and `statusStrandAddSpec`, and `start.go`'s file comment, drop the llm arm's removal (card 6 already words `ensureStatusStrand`'s comment for both drivers).
`loomcli/doc.go` states the checklist verb and the strand on both drivers; `docs/overview.md`'s two bootstrap sentences (the loom module bullet and the execution-stack bootstrap bullet) say the status strand runs the checklist and is kept on both drivers.

Tests:
- `TestRowNames_FollowEachRecipesRowOrder` (new, `recipe_test.go`): each recipe's names equal its `shedbuild.Parse` row names in order, and `batten` is refused.
- `checklist_test.go`: `TestRenderChecklist`, table-driven with a fixed `now` and `started_at`, rows asserting exact lines: every recipe row in order with the right box on a running run; a row reviewed twice with its summed time and `x2`; a row whose visits include a folded `repeats: 1` entry and a `goto` (one row covers both child drivers, since the go and llm drivers write the same status file through `shedengine` and the checklist reads only that file); a row resumed after a failed call counting the halt in its time; a row reached by `goto` after a halt counting only the time since the `goto`; a blocked run's current row halted with its time running to `now`; a failed run's current row failed; a history entry naming a removed row after the recipe rows; an unreadable seed or recipe (`rowsErr` set) showing the state line and `recipe unreadable`; an absent status showing the unavailable line and every row not reached; and rows that call the frame's `Fit` with fewer rows than lines, where the fitted lines hold the state line, the running row and the summary line, never more than the told rows, and drop the rows furthest from the running row first.
- `TestChecklistCmd_ReportsAResolutionFailureOnTheEnvelope` (new, `checklist_test.go`): `RunCLIIn` from a temp directory that is no git worktree, args `checklist`, exits 1 with an `"ok":false` envelope and starts no pane.
- `TestStatusStrandCmd` wants `loom checklist`; `TestStatusStrandAddSpec` wants the top content declaration, and its "pinned by render" sub-test wants the strand pinned at its content height both above a driver strand and alone.
- `TestSmokeStatusStrandAcrossDriverSeeds` (tmux tier): its llm-seeded start sub-test now wants exactly one status strand after the start, and the file comment says so; the step sub-tests are unchanged.
- `test-budget.yaml`: `internal/loomrecipe` 52 → 53; `internal/loomcli` loses two tests on this card and gains two, at 204 after card 6, so its row of 207 needs no raise.

**ImpactSummary:** loomcli's bootstrap and one new hidden verb plus one loomrecipe accessor; the llm-arm removal and its two tests go, and no other package calls them.
