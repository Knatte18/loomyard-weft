```yaml
lens: repo-invariants
outcome: reported
```

Lens: repo-invariants. For each card I checked test seams, test tier tags, doc text the card makes false, and whether the card can pass its gate in card order.

**Per card: each invariant, held or broken**
- **Card 1:** Seams held: no `//lym:seam` test walks `pauserecord.Coverage` or `dobbyengine.RunView`. Tiers held: every test change is a row in an existing tier0 test. Docs held: the pauserecord and dobbyengine `doc.go` are listed, and tenderengine's is under Extends. Order held.
- **Card 2:** Seams held: no seam test walks `BeamUpOptions`. Tier held: one tier0 row; the harness's `runWith` takes the options. Docs held: orchcli `doc.go` is under Extends. Order held.
- **Card 3:** Tier held: the new `record_test.go` is tier0 and `testmain_test.go` is an untagged support file. Docs broken: see finding 2. Order held: it uses only `fabricengine.AdoptScratchDir`.
- **Card 4:** Seams held: no seam test walks `greenDeps`. Docs held: the beamcli `doc.go` and `createCmd` (`internal/beamcli/cli.go:780`), the battenshed `doc.go`, `docs/overview.md` and the landingshed refusal are listed. Order held: `entryDone`, `resumeChild` and the `hold`/`holds` fixture helpers have no other test callers.
- **Card 5:** Seams held. Order held: `checkStateNone` is read in `world.deps` and `hubDepsAs`, and the card lists both. Docs: the adoptcli `doc.go` stays stale until card 10, which owns it, so held.
- **Card 6:** Order held: `holdsFileName` and `appendHolds` are read in `TestRunCheck` (`check_test.go:501,512`) and `TestFailedEnd`, and the card lists both. `heldSlugs` is deleted with its test uses. Docs: the `holds.go` file comment is rewritten, so held. The `Command` `Long` is at risk, see finding 1.
- **Card 7:** Order held: `adopt`'s callers are `cli.go`'s `Command`, `export_test.go`'s `AdoptAs` and `adopt_test.go`, and the card lists all three. Tier held: `TestRunWorker` and `TestJudgeStanding` are tier0 over `t.TempDir()`. Seam held: no seam test walks `adoptDeps` or `workerDeps`.
- **Card 8:** Docs held: the runproc `doc.go`, `PATTERN-daemon-wakeups` and `unboundedWaitFiles` are listed. Tiers held: the new tests are tier0, and `TestSpawnersStartNothingUnderTest` replaces a tier0 test in `deps_test.go`. CLI-tree guard broken: see finding 1.
- **Card 9:** Tier held: the `TestStatus` change stays in a tier1 file and `TestAdoptSectionOf` is tier0. Docs held: the beamcli `doc.go` and `statusCmd` are under Extends.
- **Card 10:** `PATTERN.md` and the `PATTERN-dobby-runs` background are listed, so held.

**Findings**

1. **MEDIUM, design. Card 8, `internal/adoptcli#Command` (`Long`). Card 6 also edits it.** The `Long` text is already about 1990 bytes against `maxLongBytes = 2000` (`cmd/lym/clitree_test.go:169`). Card 8 adds the worker, the follow, the record, `--no-orch` and `--discard` to it, and card 6 rewrites its step list. No card bounds the text to the cap, so card 8's gate fails the `clitree` walk unless the detail moves to `internal/adoptcli/doc.go`. Evidence: `internal/adoptcli/cli.go:24-54` measures 1997 bytes for the `Long:` span; `cmd/lym/clitree_test.go:167-169,278`.

2. **LOW, consistency. Card 3 (or card 10).** Two comments say `fabricengine.AdoptScratchDir` holds the adopt check's files only. The doc reads "`AdoptScratchDir(hub)` holds the adopt check's files", and the function's own comment reads "the directory that holds the adopt check's record, output and lock". Card 3 puts `adopt.json` and `adopt.lock` there, and card 8 adds `adopt.out`. No card lists either comment. Evidence: `internal/fabricengine/doc.go:1122`, `internal/fabricengine/junctionnames.go:188`.

**Tool calls**
1. Grepped for `//lym:seam` markers on the touched structs, and for every file holding hold or green-end text. My context had no list of seam tests or a repo-wide doc sweep.
2. Grepped the stale-text lines in files no card lists, plus the adopt mentions in the fabricengine doc. The manifest did not cover these files.
3. Read `fabricengine/doc.go:1118-1126` and `junctionnames.go:186-192`, grepped the tests for pinned hold texts, read the `maxLongBytes` cap and the adoptcli `doc.go` scratch list. The manifest lacked the scratch-dir docs and the CLI cap.
4. Grepped the test callers of `adopt(`, measured the `Long` text's byte length and checked the tier tags in `deps_test.go`. I needed the deletion callers and the cap headroom.
