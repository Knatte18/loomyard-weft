# Card 3 — calibration report

**Edit:**
- `internal/gitrepo/blobread.go`
- `internal/gitrepo/blobread_integration_test.go`
- `tools/tokencount/count.go`
- `tools/tokencount/main.go`

**Create:**
- `tools/tokencount/calibrate.go`
- `tools/tokencount/calibrate_test.go`
- `tools/tokencount/calibrate_integration_test.go`
- `tools/tokencount/testmain_test.go`

**Uses:**
- `plan:internal/batcher#SegmentCost`
- `plan:internal/batcher#ProfileWeights`
- `plan:internal/batcher#SizeSource`
- `plan:internal/batcher#Weights`

**Intent:**
The calibration report: the estimator run on past plans, against the tree each run started from, beside each card's measured fork weight (Decision 3).
It is read-only and calls the same `batcher.SegmentCost` the live batchifier will call.

**gitrepo, go-git local reads only** (`PATTERN-gitrepo-client-boundary`), in `blobread.go`:
- `CommitsWithSubject(subject string)` returns every commit reachable from any branch or tag whose message's first line equals `subject`, once each, with its SHA and committer time, newest first.
- `FilesInDirAtRevision(rev, dir string)` returns the names of the regular files directly in `dir` in `rev`'s tree, sorted; an absent directory is an empty result, not an error; an invalid `rev` is `ErrInvalidSHA`, as for `FileAtRevision`.
`TestHistoryReads` gains steps for both: a subject matched on two branches and a tag counted once, a subject absent, a directory listing, an absent directory.

**Run base from the transcripts** (`count.go`).
While reading a `webster` session, `CountRun` records `RunTally.BaseSHA`: the `start_sha` of the earliest `lyx webster begin-batch` result envelope in the run's `webster` session transcripts.
That is the HEAD before the run's first batch forked, so the parent of the first commit that batch made: the tree the batch started from.
Reading it from the transcript needs no `state.json` lookup, and an absent envelope leaves `BaseSHA` empty.

**Calibration** (`calibrate.go`).
Per run of the report:
1. The run's first fork time is the earliest `ForkTally.Started`; a run with no fork is skipped with that reason.
2. The plan commit is the newest commit with subject `loom: plan artifacts for <slug>` committed before that time, from the history repository; none is a skip with that reason.
3. The plan is parsed with `planparser.ParsePlanFrom(planparser.PlanDirRel(), …)`, reading each plan file at the plan commit and mapping `gitrepo.ErrPathNotAtRevision` to an error wrapping `fs.ErrNotExist`; a parse failure is a skip naming the error.
4. The base is `RunTally.BaseSHA` in the code repository (the current worktree); empty, or absent from the repository (`SHAExists`), is a skip with that reason.
5. A git-tree `batcher.SizeSource` over the base answers `Lines` through `FileAtRevision` and `TestFiles` through `FilesInDirAtRevision` filtered to `_test.go`.
6. Per card: the estimate is `SegmentCost` of the one-card segment with the named profile's `ProfileWeights`; the measured weight is the summed `Usage.Weight()` of every fork whose `Cards` names the card; the ratio is measured over estimate.
   A card no fork names is skipped with that reason.
   A card a multi-card fork names is skipped as "ran in a multi-card fork", since only one-card forks measure a one-card cost.
   A card whose estimate is 0 is skipped with that reason.

The report writes `## Calibration (<profile>)`: one row per card (run, card, estimate, measured, ratio), the skipped runs and cards each with its reason, and a fit summary per run and overall: the number of cards, the median ratio and the spread, the 75th percentile of the ratios over the 25th.
Percentiles interpolate linearly between the closest ranks.

**Flags** (`main.go`): `-calibrate <profile>` adds the calibration section; `-history <dir>` names the repository holding the plan commits, required with `-calibrate`; `-config <dir>` names the directory whose `batcher.yaml` holds the profile, default the current directory.
The code repository is the current directory, through `lyxcwd.Getwd` as today.
The package comment describes the calibration section, the lookups and every skip reason.

**Tests.**
`calibrate_test.go` (untagged) drives the calibration over an in-memory history and size source and a fixture `batcher.yaml` seeded under `t.TempDir()`: exact estimate (equal to `batcher.SegmentCost` on the same inputs), measured weight and ratio for an attributed card; a card two forks name takes their sum; skips for a run without a plan commit before its first fork, a run without a base, a card no fork names, a card a multi-card fork names; the per-run and overall median and spread.
`calibrate_integration_test.go` (`integration`) builds one repository with `gitkit` holding a plan commit and the base tree, and asserts the gitrepo-backed history and size source give the same row as the in-memory ones.
`testmain_test.go` declares `TestMain` calling `gitkit.HermeticGitEnv()` and running through `tmuxkit.Main` (`PATTERN-test-isolation`).
`tools/` test files never name either fabric side (`PATTERN-fabric-vocabulary`).

**ImpactSummary:** `internal/gitrepo` gains two read-only go-git methods no production caller outside `tools/tokencount` uses; `tools/tokencount` gains the calibration section and three flags, its existing output unchanged without `-calibrate`.
