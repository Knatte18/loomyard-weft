# Card 4 — records-replay

**Create:**
- `plan:internal/fabricengine#EndedSet` -> `type EndedSet map[string]string`
- `plan:internal/fabricengine#readEndedSet` -> `func readEndedSet(recordsRoot, boltRef string, slugs []string, localCreated map[string]string) (EndedSet, error)`
- `plan:internal/fabricengine#finalizeEntered` -> `func finalizeEntered(l *lyxcwd.Location, slug string) bool`
- `plan:internal/fabricengine#ReplayOutcome` -> `type ReplayOutcome struct`
- `plan:internal/fabricengine#ErrReplayConflict` -> `var ErrReplayConflict error`
- `plan:internal/fabricengine#ErrReplayDirty` -> `var ErrReplayDirty error`
- `plan:internal/fabricengine#replayWeft` -> `func replayWeft(rec *Mutations, l *lyxcwd.Location, worktree, upstream string, ended EndedSet) (ReplayOutcome, error)`
- `plan:internal/fabricengine#replayBolt` -> `func replayBolt(rec *Mutations, l *lyxcwd.Location, boardDir, upstream string, ended EndedSet) (ReplayOutcome, error)`
- `plan:internal/fabricengine#cherryPickOnto` -> `func cherryPickOnto(rec *Mutations, req pathRequest, repo *gitrepo.Repo, onto string, commits []string) (dropped []string, err error)`
- `plan:internal/fabricengine#setAsideTag` -> `func setAsideTag(rec *Mutations, repo *gitrepo.Repo, slug, sha string) (string, error)`
- `plan:internal/fabricengine#ReplayWeftForTest` -> `func ReplayWeftForTest(l *lyxcwd.Location, worktree string, ended map[string]string) (ReplayOutcome, error)`
- `plan:internal/fabricengine#ReplayBoltForTest` -> `func ReplayBoltForTest(l *lyxcwd.Location, boardDir string, ended map[string]string) (ReplayOutcome, error)`
- `plan:internal/fabricengine#ReadEndedSetForTest` -> `func ReadEndedSetForTest(recordsRoot, boltRef string, slugs []string, localCreated map[string]string) (map[string]string, error)`
- `plan:internal/fabricengine_test#TestRecordsReplay` -> `func TestRecordsReplay(t *testing.T)`
- `plan:internal/fabricengine#KindTagCreated` -> `const KindTagCreated Kind = "tag_created"`

**Edit:**
- `internal/fabricengine_test#manifestObservableKind`

**Uses:**
- `internal/fabricengine#dropSeedCommits`
- `internal/fabricengine#Fabric.HoldsWork`
- `internal/fabricengine#ReadOriginFor`
- `plan:internal/fabricengine#InstallBoardReplay`
- `plan:internal/testkit/boardkit#InstallReplay`

**Intent:** D5's records replay and D3 step 2's ended-set rule, one fabricengine helper pair (new file `replay.go`) that the pull, push and reconcile cards call.

`readEndedSet` reads, from the records repo's ref `boltRef` (origin's Bolt tip as last fetched) and that tip's own history only, every slug of `slugs` whose entry is `done` on the tip, or absent from the tip and present in a commit of the tip's history; where `localCreated` holds a `created` stamp for the slug, the historic entry must carry that same stamp, so a pruned slug reused by an entry only local Bolt holds is never ended.
It reads `board.json` through the installed `BoardReplay.Entries` and runs no network call; a slug only local Bolt ever held matches nothing and is not ended.
`EndedSet` maps a slug to `done` or `removed`.
`finalizeEntered` reports whether this machine's local state shows the run's Finalize past its parent-side merge, the step after which Finalize writes the done mark and its landing reaches origin, read from three places, decoding only the producer names and tolerating unknown keys:
the task pair's loom status at `shedrun.StatusFile` of the pair's location, when the pair is present, whose history holds a `Finalize` row;
the same status whose `current_producer` is `Finalize` with no such row, only when the local parent branch, named by the pair's origin record, already holds the task branch's work by `Fabric.HoldsWork(<local parent tip>, <task head>)`, which Finalize's own parent-side merge makes true before it marks the entry done;
and the prime's batten run status of the slug, whose `current_producer` or history names `Land` or a later row, or whose state is `done`, since batten reaches Land only after its child's Finalize and keeps writing the slug's records on the prime weft after Worktree-Teardown removed the pair.
`current_producer` `Finalize` alone never counts, because Shed persists it before it calls the row, so it reads `Finalize` at every Finalize entry; a Finalize that entered without merging here, with an entry done on origin, is a landing made elsewhere.
A local parent fast-forwarded to another machine's landing also holds the work, and a Finalize then lands nothing new, through its already-landed path.
A read failure of either status or of the parent tip counts as entered, logged at Warn, so a doubt never sets aside this machine's own landing.
Both replays drop such a slug from the ended set, so this machine's own landing is never set aside.

`replayWeft` brings a weft worktree's branch level with `upstream` when both moved, under its caller's locks.
It refuses with `ErrReplayDirty`, naming the paths, a tracked change on a path the replayed commits touch, changing nothing.
It sets aside each local commit origin lacks whose every path lies under an ended run's record directory (`shedrun.RunDirRel`), putting a local `set-aside/<slug>/<sha>` tag on it (`setAsideTag`, a prefix no archive reader matches, which appends a `KindTagCreated` entry to `rec` only after the tag exists, Target the repo root, Detail the tag name), then replays the other commits oldest first onto `upstream` by cherry-pick through `cherryPickOnto`, dropping a commit that becomes empty.
`cherryPickOnto` lives in `destroy.go` beside `dropSeedCommits` and reuses its executor shape (`ResetKeep` to `onto`, cherry-pick, and on a conflict `CherryPickAbort` then `ResetKeep` back to the starting tip), per PATTERN-fabric-destruction-chokepoint; a conflict returns `ErrReplayConflict` wrapped with the conflicting paths, the starting tip restored.
`replayBolt` does the same for Bolt, with two differences: a commit is set aside when every changed `board.json` entry and every `design-<slug>.md` it touches belongs to an ended entry (`README.md` not counted); and a kept commit is re-applied by entry through the installed `BoardReplay.Apply` onto origin's tip, so `README.md` and an entry only one side changed never conflict.
A Bolt commit `Apply` reports `Empty` is dropped with no re-render; one that lost a value or met a refusal is committed for what was kept and its original is kept under a `set-aside/<slug>/<sha>` tag, reported with the lost values and refusals; no entry change stops a Bolt replay.
A commit touching Bolt paths outside the board (stencils, config) is replayed by cherry-pick as on a weft, and a seed commit among them that conflicts is dropped as `dropSeedCommits` drops it today.
`ReplayOutcome` reports the replayed, dropped and set-aside commits with their tags, the conflicting paths, and the lost values and refusals.
`KindTagCreated` is the one new mutation kind, a local tag write, distinct from `KindTagPushed`'s pushed archive tag; `manifestObservableKind` classifies it false, a git-state kind.
Bound: it replays only records branches, only commits origin lacks, sets aside only an ended entry's commits, never rewrites a commit origin holds.

The three `ForTest` shims in `export_test.go` expose the helpers to `TestRecordsReplay` (integration, a `hubforge` hub and a second clone of its records bare as the other machine, `boardkit.InstallReplay` called first), with rows: a local status commit under newer unrelated origin commits replays, an empty commit dropped; a `status.json` changed on both sides stops with `ErrReplayConflict` naming it, tip restored; a tracked change on a touched path refuses with `ErrReplayDirty` and moves nothing; Neve's case, local Bolt and prime-weft status commits of an entry done on origin set aside under `set-aside/` tags and the rest replayed, the replayed prime-weft tip holding no run directory of the slug and no set-aside tag matching `fabricengine.ArchiveTagPrefix`, the two sources `lyx report`'s run listing reads (`internal/reportengine/source.go`), so it lists no run for them; an entry pruned on origin is ended even after local Bolt fast-forwarded past the removal; an entry only local Bolt holds is not ended; a reused slug with a different `created` stamp is not ended; a slug whose pair status shows `Finalize` in its history is not set aside, nor one whose pair status has `Finalize` as its current producer with no history row while its local parent branch holds the task's work, nor one whose pair is gone and whose prime batten status stands at `Land`; a slug whose pair status has `Finalize` as its current producer with no history row and a local parent branch not holding the task's work, its entry done on origin, is ended and its commits set aside; two machines' Bolt status commits of different entries replay with no stop and `README.md` re-rendered; one field changed on both sides keeps origin's value and tags the local commit; a local Bolt body edit of an entry origin holds under a run status is kept under a `set-aside/` tag and reported with the run lock's refusal, and the replay does not stop; a local Bolt commit touching a stencil file origin left alone is replayed by cherry-pick; a local seed commit conflicting with origin's change of the same stencil file is dropped as `dropSeedCommits` drops it.

**ImpactSummary:** One new mutation kind, `KindTagCreated`, and one row in the mutation oracle's `manifestObservableKind` classifying it; the rest is new code no existing caller reaches.

**Commit:** 4: fabricengine: records replay and the ended-set rule
