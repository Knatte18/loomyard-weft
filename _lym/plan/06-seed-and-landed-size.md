# Card 6 — seed-and-landed-size

**Create:**
- `internal/fabricengine#DiffSize`
- `internal/fabricengine#Fabric.DiffSize`
- `internal/fabricengine_test#TestFabric_DiffSize`
- `internal/loomcli#runUsageRecord`
- `internal/loomcli#TestRunUsageRecord`

**Edit:**
- `internal/loomcli#loomCLI.seedAndCommitBootstrap`
- `internal/loomcli#landingDeps`
- `internal/landingshed#Deps`
- `internal/landingshed#Finalize`
- `internal/landingshed#Finalize.mergeParent`
- `internal/landingshed#Finalize.recordLanding`
- `internal/landingshed#TestFinalize_RecordsTheLandingCommitOnTheFirstTry`
- `internal/landingshed#TestFinalize_RecordLandingFailureKeepsDone`
- `internal/landingshed#recordingParentMerger`

**Uses:**
- `internal/fabricengine#Fabric.LineChange`
- `internal/fabricengine#countNumstatLines`
- `internal/shedrun#WriteRunUsage`
- `internal/shedrun#WriteLandedSize`
- `internal/shedrun#UsageDir`
- `internal/runprofile#Load`
- `internal/boardengine#OpenHub`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/fabricengine/doc.go`
- `internal/landingshed/doc.go`

**Intent:** D12's run-level records, which D13's cells key on.
Seed: `seedAndCommitBootstrap`, on the run's first seed, writes `run.json` into `shedrun.UsageDir` through `shedrun.WriteRunUsage`, built by `runUsageRecord`: the run's recipe, its profile (the entry's `profile` field, else the recipe's entry under `profiles.yaml`'s `defaults`, read with `runprofile.Load`), and `stored_record`, true when the run's board entry is flagged discussed at the start, so the run starts from a stored record (D3); an absent entry writes an empty profile and false.
A re-entry that finds `run.json` already there keeps it.
The file joins the run records through card 5's status-commit pathspec.
A failed read of the board or of the profiles is logged at Warn and writes the record with what was read, so the bootstrap never fails on it.
Finalize: `fabricengine.Fabric.DiffSize` reads `git diff --numstat -M from to` in the pair's code repository, as `LineChange` does, into `DiffSize{LinesAdded, LinesDeleted, FilesChanged}`, a binary file counting as one file and no lines.
`landingshed.Deps` gains `RecordLandedSize func(from, to string) error`, nil writing nothing.
The landed change is the task's own change past the parent: once `mergeParent` has merged the parent into the task, the parent head it merged (the parent pair's `HeadSHA()` read right before that merge) is the base, and the landing commit is the tip, so `from` is that parent head and `to` the landing commit.
`mergeParent` keeps that parent head on a `Finalize` field, and `recordLanding`, after `RecordLanding`, calls `RecordLandedSize(thatHead, landingCommit)`; a Finalize resumed in a new process past the merge holds no such head and skips the record, logged at Info, so that run enters its cost cell's means only, a bounded loss on a rare path; a failure is logged at Warn and never changes the verdict, as `RecordLanding`'s is.
`landingDeps` fills `RecordLandedSize` with the pair's `DiffSize` written through `shedrun.WriteLandedSize` into the worktree's usage directory, so the landed record rides the run's last status commit.
`recordingParentMerger` gains a field scripting the parent head read before the merge apart from the landing head.
`TestRunUsageRecord` (tier0) has rows: an entry with its own profile, an entry with none taking the default, a discussed entry (`stored_record` true), an absent entry.
`TestFabric_DiffSize` (tier1, a hermetic repository) has rows: a change adding and deleting lines in two files, a binary file, a pure rename.
`TestFinalize_RecordsTheLandingCommitOnTheFirstTry` gains the assertion that `RecordLandedSize` gets the scripted pre-merge parent head and the landing commit, which differ; `TestFinalize_RecordLandingFailureKeepsDone` gains a row where `RecordLandedSize` fails and the verdict stays Done, and a row with no merged head held, where it is not called.

**ImpactSummary:** One new optional Finalize seam and one bootstrap write that never fails the bootstrap; `TestWire_EverySeamFilled` checks the new seam is filled by `landingDeps`.
