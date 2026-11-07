```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. I list, per change, what the change forces outside the lines it touches, and whether the diff covers it.

**List: per change, the set found, covered or not**

1. **`VerbCase.Build` field and the Build/Arrange split of five cases**
   - Constructors:
     - `addCase`, `reconcileCase`, `pullCase`, the two CloneHub{Reset} cases and the hostile cases leave `Build` nil.
     - Their `Arrange` stays whole, which matches D7: `Add` keeps adding its pair, `Pull` pushes through a `tb.TempDir()` clone, and the CloneHub{Reset} cases place directories outside the hub. Covered.
   - Readers of `Arrange` that bypass `copyVerbHub`:
     - `livestate_matrix_test.go:382-383` is the CloneHub{Reset} column, which uses `CopyHub` plus `Arrange`.
     - None of those cases has a `Build`, so nothing is skipped today. Covered; no finding.
     - `livestate_verbs_selftest_test.go:41` is moved onto `copyVerbHub`. Covered.
   - Docs: `livestate_doc_test.go` (header, Cleanup protection and omission paragraphs), the `VerbCase` doc comment, the file header, the `Omissions` reasons and the per-case comments are updated. One miss, F1 below.
   - Registrations:
     - `verbTemplates` initialises from `Verbs`; Go's dependency ordering covers this.
     - Keys are `fabricengine-livestate-&lt;Name&gt;-&lt;anchor&gt;`, and each `Registration` is created once, so one address per key.
     - The `Build` closures capture only the `const slug`, so they carry no per-call data. Covered.
   - Cleanup `OriginalBranch`:
     - It now reads `BoardDir()` after the build instead of `PrimeWorktree()` before it.
     - `BoardDirName == BoltDirName` (`junctionnames.go:113`), and `primaryWeftBranch` (`cleanup.go:337`) reads that same checkout's branch.
     - The value is the same by construction. Covered; no finding.
   - Build safety:
     - `mustGit` (`livestate_helpers_test.go:15`) panics instead of using the `tb`.
     - Inside a registered template build, which runs on its own goroutine (`template.go:184`), a git failure crashes the test binary instead of being recorded as the template's failure.
     - This is only a failure-path degradation. The old Arrange path panicked in the same way, and no process-wide state is touched. Not a PATTERN violation; no finding.

2. **`commitOnBranch`, `branchAtCurrentHEAD`, `setupConflictingDivergence` and `setupCleanNonFastForward` change from `*testing.T` to `testing.TB`.** Every caller passes a `*testing.T`, which satisfies the interface. Covered. `setupCleanFastForward` stays `*testing.T`, which is fine because it is not used in a build.

3. **Six merge layers and `newMergeLayerFixture`**
   - Replaced sites (all steps reproduced exactly, in the same order):
     - `conflictLayer`: mergein ×3, recovery ×1, mergestage ×2, weftguards ×2.
     - `clashLayer`: recovery ×3, mergelock ×1, weftlocal ×1.
     - `nonFastForwardLayer`: recovery ×3.
     - `warpProgressLayer`: weftguards ×3.
     - `featureBothSidesLayer`: crucible ×5, weftguards ×2.
     - `cleanMergeableLayer`: mergelock ×2 sites (one is the `newCleanMergeablePair` helper).
   - Commit-message claims: "six layers" and the file-name lists match the diff. The claim "repeated at least three times" for `cleanMergeableLayer` rests on a helper that several tests call, with only two source sites. I treat it as borderline and not a defect.
   - Sites with the same steps left unmoved: F2 below.
   - Other sites that look similar but differ, so they correctly stay:
     - `weftguards_integration_test.go:98`: no weft counterpart branch.
     - `mergestagetracked_integration_test.go:32`: seeds `tracked.txt` first.
     - `mergesiblings_integration_test.go:72`: takes a fork SHA first and uses `RecordsBranchName`.
     - `mergein_integration_test.go:240`: a different weft commit.
     - `mergein_integration_test.go:549` and `:486`: other file names or an extra step.
     - `mergelock_integration_test.go:177` and `:244`: a different weft commit (`_lym/clean.txt`).
     - `mergein_recovery_integration_test.go:1208`: a decoy branch is built first.
   - Process-global seams: the grep for `Setenv`, seam setters and `.Context()` across the merge test files found none in the changed files. The `installFailingPreCommitHook` call acts on a copy's own repository. Covered.
   - Ordering change: `OpenFabric` now runs on a hub whose branch commits already exist, where it used to open before them. I found no code in my context that reads branch state at open time. Not covered by anything I read, and I raise no finding.

4. **Declared PATTERN changes, `PATTERN-test-isolation`, `PATTERN-test-speed`.** The diff touches no PATTERN file. The D7 constraints are for this package only: no process-wide builds (the clean builds above), no wrapper fixture type (`mergePairFixtureOf` and `newMergeLayerFixture` are plain helpers over `*hubforge.Hub`), and a `Registration` per layer. No pattern departure trailers appear in the four commit messages. Covered; no finding.

**Findings**

- **F1 — LOW, `consistency`, `internal/fabricengine/livestate_verbs_test.go:664-665`.** A stale comment inside the Cleanup `Expect` says "Arrange moves the prime pair off f.OriginalBranch". That step now lives in `Build` (`:605`, `:635`), and the commit updated the neighbouring comments to `Build`.
- **F2 — LOW, `scope`, `internal/fabricengine/midmerge_integration_test.go:41-43`.** The site repeats the `clashLayer` steps exactly (`setupConflictingDivergence(... "feature","clash.txt")` then `branchAtCurrentHEAD(... "feature-weft")`) after `newMergePairFixture(t, ".")`. It is the only identical site left unmoved, and the commit says "Sites whose commits are not repeated at least three times keep their own steps". The layer exists and this site is its sixth user, so it should use `newMergeLayerFixture(t, &amp;clashLayer)`.

**Tool calls (2)**
1. A combined read of `livestate_helpers_test.go` (`mustGit`), a grep of "Arrange" in the live-state tests, and slices of the unmoved merge sites: needed to check the stale comments and the identical sites, which my context lacked.
2. Slices of `mergesiblings`, `mergein` and `mergein_recovery`: to confirm the remaining `setupConflictingDivergence` and `setupCleanNonFastForward` sites differ from the layers.
