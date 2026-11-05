# Card 12 — beam-describe-verb

**Create:**
- `plan:internal/fabricengine#LineCommit` -> `type LineCommit struct`
- `plan:internal/fabricengine#BeamLineCommits` -> `func BeamLineCommits(l *lymcwd.Location, parentBranch string) ([]LineCommit, error)`
- `plan:internal/fabricengine#TestBeamLineCommits` -> `func TestBeamLineCommits(t *testing.T)`
- `plan:internal/beamcli#beamDescribeDeps` -> `func beamDescribeDeps(location *lymcwd.Location, slug, parentBranch, recordPath string) landingshed.BeamDescribeDeps`
- `plan:internal/beamcli#describeCmd` -> `func describeCmd() *cobra.Command`
- `plan:internal/beamcli#describe` -> `func describe(deps landDeps, slug string) (map[string]any, error)`
- `plan:internal/beamcli#judgeAwaitingRun` -> `func judgeAwaitingRun(deps landDeps, verb, slug string) error`
- `plan:internal/beamcli#judgeNotReady` -> `func judgeNotReady(deps landDeps, verb, slug string) error`
- `plan:internal/beamcli#TestDescribe` -> `func TestDescribe(t *testing.T)`
- `plan:internal/beamcli#TestHubLandDeps_DescribeStampsAndCommits` -> `func TestHubLandDeps_DescribeStampsAndCommits(t *testing.T)`

**Delete:**
- `internal/loomcli/beamwiring.go`
- `internal/loomcli/beamwiring_integration_test.go`

**Edit:**
- `internal/beamcli#Command`
- `internal/beamcli#landDeps`
- `internal/beamcli#hubLandDeps`
- `internal/beamcli#newLandWorld`
- `internal/beamcli/doc.go`
- `internal/loomcli/doc.go`
- `internal/loomcli/pawlwiring.go`
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/landingshed#WriteBeamDescription`
- `internal/landingshed#BeamDescribeDeps`
- `internal/pawlengine#UpdateLanding`
- `internal/loomengine#BeamLandingPath`
- `internal/loomengine#BeamLandingLockPath`
- `internal/loomengine#ProjectDecisionRecord`
- `internal/loomengine#LandingDirRel`
- `internal/beamcli#judgeLandEntry`
- `internal/beamcli#finishVerb`
- `internal/beamcli#onHubLandDeps`

**Intent:** Add `lym beam describe <slug>` (D5): the orch asks for the beam's change description, and nothing at landing rewrites it.

- The beam's own line moves into fabricengine (`PATTERN-fabric-git`), so beamcli runs no git itself: `fabricengine.BeamLineCommits` lists the commits on the beam pair's first-parent line that its merge base with parentBranch lacks, oldest first, as loomcli's `beamCommits` does today over `gitrepo`, refusing an empty parentBranch with today's way forward. Each `LineCommit` carries `SHA`, `Message`, `Committed` and `RecordsOnly`, true when every path the commit changes against its first parent, read through `gitrepo`'s `CommitDetail`, lies under `lymdirs.LymDirName`: lym's own records (the run's status commits, the description, the landing and ready records), never a landing of work. `internal/loomcli/beamwiring.go`, uncalled since card 5, and its integration test go; `TestBeamLineCommits` carries that test's case over a hermetic hub and adds a records-only commit. `internal/loomcli/pawlwiring.go`'s header drops "beyond the BeamDescribe row", since card 5 removed that row.
- `beamcli.beamDescribeDeps` builds the description writer's deps over the beam pair, as loomcli's did: its `Commits` lists `BeamLineCommits` without the records-only ones, so a describe never lists its own commits or the run's status commits; `Commit` commits the landing directory with the message `beam: change description for <slug>`.
- `judgeAwaitingRun` refuses, naming `lym batten status <slug>`, a beam with no pawl run (naming `lym beam create <slug>`) and one whose run is not at Beam-Await; a run at Beam-Await passes in every state (running, paused, failed, blocked and awaiting), so a halted run is served, since Beam-Await halts by an error stop the status records as failed. `judgeNotReady` refuses a beam whose ready record names it, saying `lym beam ready` is final and the landing is under way, with the way forward `lym batten status <slug>`; a failed read refuses naming the error. Cards 13 to 15 reuse both.
- `describe` runs `judgeLandEntry`, `judgeAwaitingRun` and `judgeNotReady`, then the `Describe` seam: write `summary.md` whole through `landingshed.WriteBeamDescription`, stamp the landing record's `DescribedAt` through `pawlengine.UpdateLanding`, and commit the landing directory, the description and the record together. A validation finding is refused with the finding and `lym board upsert` for the entry's title or brief, as `WriteBeamDescription` words it. The envelope names the description's path and the way forward: `lym beam pr open <slug>`, or `lym beam pr update <slug>` when a pull request is recorded.
- `landDeps` gains `Describe func(slug string, at time.Time) (path string, err error)`; `hubLandDeps` fills it over the beam's pair: `beamDescribeDeps(beam, slug, origin.ParentBranch, loomengine.ProjectDecisionRecord(beam, slug))` and `WriteBeamDescription`, then the stamp at `loomengine.BeamLandingPath`/`BeamLandingLockPath`, then one `fabricengine.CommitAnchoredPaths` over `loomengine.LandingDirRel()`.
- `describeCmd`'s help says the verb owns `summary.md` and writes it whole each run from the entry's title and brief, the beam's first-parent commits with their pull request numbers or commits, merge times and summaries, and the project record's added decisions; that the orch never edits the file by hand and changes its text only through `lym board upsert`; and its refusals. `Command` adds it, and its Short and Long name it.
- Bound: the verb writes only the beam's own landing directory, commits only there, and refuses once the ready record stands, so nothing at or after landing rewrites the description.

Tests: the new `TestDescribe` (tier 0) drives `describe` over `newLandWorld`'s fakes, its passing rows first clearing the fixture's default ready flag (`w.ready["b"] = false`): a beam at Beam-Await running and one failed are described, the `Describe` seam called once with the slug and the envelope naming the path; refusals for an entry that is no beam, a beam with no run, a run at Finalize, a run at Beam-Down, a standing ready record and a failed ready read, each naming its way forward; a `Describe` error carrying a validation finding is returned naming `lym board upsert`. `newLandWorld` gains the `Describe` fake. The new `TestHubLandDeps_DescribeStampsAndCommits` (tier 1) runs `hubLandDeps(prime).Describe` over a hub fixture with a beam pair holding a member landing and a records-only commit: `summary.md` lists the landing and not the records-only commit, the landing record names the beam with `DescribedAt` set, and one commit holds both files. `TestBeamLineCommits` (tier 1) as above.

**ImpactSummary:** one new verb in beamcli over a new fabricengine line read; loomcli loses a file and its test that no code there calls since card 5.

**Commit:** 12: Add lym beam describe
