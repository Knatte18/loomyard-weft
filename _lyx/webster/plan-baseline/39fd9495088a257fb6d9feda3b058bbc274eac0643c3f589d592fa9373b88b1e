# Card 4 — conflict-session-skills

**Create:**
- `plan:internal/landingshed_test#TestMergeIn_ConflictSessionLoadsLanguageSkills` -> `func TestMergeIn_ConflictSessionLoadsLanguageSkills(t *testing.T)`

**Edit:**
- `internal/mergeresolve#Deps`
- `internal/mergeresolve#buildConflictSpec`
- `internal/mergeresolve#TestBuildConflictSpec_SkillsAndParentDirective`
- `internal/mergeresolve#mergeresolveAllowedImports`
- `internal/mergeresolve/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`
- `internal/landingshed#Deps`
- `internal/landingshed#NewMergeIn`
- `internal/landingshed#NewFinalize`
- `internal/landingshed_test#newFinalizeAt`
- `internal/landingshed_test#TestFinalize_OverRealHub`
- `internal/loomcli#landingDeps` -> `func landingDeps(l *lyxcwd.Location, recipe string, geom websterengine.Geometry, taskBranch, originURL, parentBranch string, pushSkipped bool, pushBranch func() error, registry modelspec.Registry, runner *shuttleengine.Runner, cfg landingshed.Config, parentName string, languageSkills []string, verifyWaitMark func(label string, start time.Time) error, stopConflictSession func() (string, error), verify verifySource, lock landingshed.ParentLockDeps) landingshed.Deps`
- `internal/loomcli#loomCLI.loomPreRun`
- `internal/loomcli#loomCLI.buildLoomShed`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/loomcli#TestLandingDeps_ConfigNoticeSeamsOverRealHub`
- `internal/loomcli#markDoneFixtureDir`
- `internal/loomcli#TestWire_Real_PublishRejectedPushNamesRemoteTip`
- `internal/loomcli#TestLandingDeps_VerifyCommandReadsPlanAtCallTime`

**Uses:**
- `plan:internal/languageconfig#AppendSkills`

**Intent:**
The merge conflict session loads the told language list after `conflictSkills`, at Merge-In and at Finalize (D4).
`landingshed` loads no config: the list arrives as a told value.

`mergeresolve.Deps` gains the field `LanguageSkills []string`, told by the caller, empty for none.
`buildConflictSpec` sets `Skills` to `languageconfig.AppendSkills(conflictSkills, deps.LanguageSkills)`.
`mergeresolveAllowedImports` admits `github.com/Knatte18/loomyard/internal/languageconfig`, its doc comment adding that the package calls only `AppendSkills`, a pure list merge that reads no file and carries no geometry, so `TestToldGeometryInvariant_AllowlistOnly` keeps passing.
`internal/mergeresolve/doc.go`'s skills sentence adds that the conflict session loads the told `Deps.LanguageSkills` after `scribe:prose` and `scribe:code-quality`.
`landingshed.Deps` gains the field `LanguageSkills []string`, documented as passed through unchanged to the resolver both producers construct, and `NewMergeIn` and `NewFinalize` each pass `deps.LanguageSkills` into the `mergeresolve.Deps` they build.

`internal/loomcli`'s `landingDeps` gains the parameter `languageSkills []string` after `parentName` and sets `landingshed.Deps.LanguageSkills` from it, still doing no I/O, so `TestLandingDeps_EveryFieldPopulated`'s reflection walk stays green in this card's commit.
`loomPreRun` and `buildLoomShed` pass `nil` for now; card 5 tells the loaded list.
The test callers `TestLandingDeps_ConfigNoticeSeamsOverRealHub`, `markDoneFixtureDir`, `TestWire_Real_PublishRejectedPushNamesRemoteTip` and `TestLandingDeps_VerifyCommandReadsPlanAtCallTime` pass `nil`.
`TestLandingDeps_EveryFieldPopulated` passes `[]string{"scribe:golang-testing"}`, so its reflection walk keeps every field non-zero, asserts `deps.LanguageSkills` equals it, and its doc comment drops the stale "sixteenth field" count for "a field added later".

`TestBuildConflictSpec_SkillsAndParentDirective` gains a per-row language list and wanted skills: the two existing rows tell none and keep wanting `scribe:prose,scribe:code-quality`;
a new row tells `scribe:golang-testing`, `scribe:code-quality` and wants `scribe:prose,scribe:code-quality,scribe:golang-testing`.
`newFinalizeAt` tells `deps.LanguageSkills` `scribe:golang-testing`, and `TestFinalize_OverRealHub`'s "resolves a conflict and squash merges into the parent" subtest asserts the conflict session's spec, `shuttle.Specs[0].Skills`, equals `scribe:prose`, `scribe:code-quality`, `scribe:golang-testing`, which pins the Finalize pass-through on a real merge.
`TestMergeIn_ConflictSessionLoadsLanguageSkills` (in `internal/landingshed/publish_integration_test.go`, `integration`-tagged, package `landingshed_test`) pins the Merge-In pass-through on a real pair, set up as `TestPublish_MergesInCleanlyBeforeCreatingPullRequest` sets up its pair:
the task branch and `main` each commit `conflict.txt` with different content, `newMergeInDepsAt` builds the deps, `deps.Shuttle` is `resolutionShuttle(taskWorktree, "resolved content\n", "conflict.txt")` and `deps.LanguageSkills` is `scribe:golang-testing`;
`NewMergeIn(deps)` reaches `Done`, and the test asserts exactly one conflict session ran and its spec's `Skills` equal `scribe:prose`, `scribe:code-quality`, `scribe:golang-testing`.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/landingshed` by one for it.

**ImpactSummary:** one optional field on each of two deps structs, one spec site and one allowlist entry; the two producer constructors copy it, `landingDeps` gains one parameter whose callers are all in `internal/loomcli` and whose production callers tell `nil` until card 5, and every other `landingshed.Deps` and `mergeresolve.Deps` literal compiles unchanged with an empty list.
