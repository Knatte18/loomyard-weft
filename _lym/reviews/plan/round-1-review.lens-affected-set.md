```yaml
lens: affected-set
outcome: reported
```

Affected-set lens: what each card's change forces outside the card, and which cards cover it.

**Per change: the set found, and the cards that cover it**

- **`ScopedPathspec` move (card 2).**
  - Callers: `internal/fabriccli/weft_verbs.go:112`, `internal/fabricengine/stencilcommit.go:73` and `status.go:161` (`detectWarpPollution`). Card 2 covers them.
  - `pattern/PATTERN-fabric-boundary.md:58-59`: card 2 covers it.
  - `TestScopedPathspec` (`internal/fabricengine/fabric_test.go:159`): no card moves it. See F3.
- **Structural dirs and `classifyPaths` (card 2).**
  - Code and tests: `commit.go:156`, `junctionnames.go:26/32/60/254/259/306`, `template_test.go:92-101` (`TestConfigTemplate`) and `structuraldirs_test.go:17` (`wiredNamesFromConfig`). Card 2 covers them.
  - Docs: `pattern/PATTERN-lym-files.md:36` and `docs/overview.md:152,206`. Card 2 covers them.
  - Comments: `junctionnames_test.go:253-254` and `structuraldirs_test.go:1,25,28` (the file header and the `TestStructuralNameSets` comments). Card 2 covers only the comment of `TestStructuralNameSets_ForAConfigNamingNeitherStructuralDirectory`. See F5.
- **Hub paths (card 3).**
  - Unexported names: `weftLockDirName` readers (`reconcile.go:338`, `recordsgit.go:42/94`, `origin.go:129/132`, `export_test.go:512`), `teardownLockDirName` (`pendingteardown.go:143`), `worktreeStagingPrefix` (`destroy.go:2141/2160`, `destroy_test.go:1137-1140`) and `stencilsDirName` (`stencilhistory.go:29`, in `StencilBaseByStamp`). Card 3 covers them.
  - Owner map and docs: `geometryTokenOwners` (`internal/lymcwd/enforcement_test.go:236-241`) and `docs/overview.md:151,209,210`. Card 3 covers them.
  - Not covered by any card:
    - `TestEnforcement_BoltDirNameDeclaredOnce` (`enforcement_test.go:526-575`): see F1.
    - `TestHubScratchAndLogsDir` (`hubscratch_test.go:30`): see F2.
    - The `_bolt` row comment at `enforcement_test.go:229`: see F6.
- **Ref names (card 4).**
  - `seedSubjectPrefix` reader (`stencilcommit.go:135`, `IsSeedCommit`) and `weftnameImportOwners` (`enforcement_test.go:580`): card 4 covers them.
  - Not covered: `TestSeedCommitSignature`'s seed and sync subject rows (`seedsignature_test.go:26-27`): see F3. The namespace sentence in `pattern/PATTERN-single-repo-hub.md:11`: see F4.
- **Mutation record value (card 6).**
  - Value-typed fields: `internal/fabriccli/anchor.go:65`, `fabriccli/envelope_test.go:38,111,163`, `internal/pairteardown/teardown.go:573` and `internal/hubforge/hub.go:223`. Card 6 covers them.
  - The first-field guard `cmd/lym/destructiveguard_test.go:253-291`: card 6 covers it.
- **Origin types (card 7).**
  - `stopWayForward` callers: `originreconcilemoves.go:421-495` (`originMover.move`), `originreconcile.go:927` (`judgeAgainstOrigin`) and `export_test.go:933`. Card 7 covers them.
  - `adoptSubject` callers: `originreconcile.go:356` (`Summary`) and `972` (`classifyEntries`). Card 7 covers them.
  - `cleanupRerun` and `rerunsCleanup`: only `Summary` reads them (`originreconcile.go:348-349`). Card 7 covers them.
- **Push types (card 8).**
  - `pairSlug` caller: `failedpush.go:179` (`recordPushFailure`). `isZero` callers: `takeover.go:136` (`takeoverStop`) and `pushclassified.go:884` (`retryWeftPush`). Card 8 covers them.
  - `PushResult` guard row (`destructiveguard_test.go:175`): card 8 covers it.
- **Verb results (card 10).**
  - Guard rows `destructiveguard_test.go:164-178`, the seam lines `mergevocab_test.go:221-222` and `pattern/PATTERN-pair-remote.md:98`: card 10 covers them.
  - Not covered: `TestCommitResult_Committed` (`committed_test.go:16`): see F3.
- **Read types (card 11).** The `DiffResult` row (`destructiveguard_test.go:192`): card 11 covers it.

**Findings**

**F1 — BLOCKING, consistency, `_lym/plan/03-layout-hub-paths.md` (TestHubPaths).**
Card 3 plans `TestHubPaths` wants as literal paths "such as `/h/_bolt/_lym/specs`". `TestEnforcement_BoltDirNameDeclaredOnce` walks every Go file, tests included (`Filter: scankit.All`). It fails on `_bolt` as a whole token anywhere outside `internal/lymdirs/dirs.go`, and `/h/_bolt/...` matches its pattern.
- Evidence: `internal/lymcwd/enforcement_test.go:526-575`.
- Fix: build each want from `lymdirs.BoltDirName` joined with the literal segments, as card 2 does for `TestClassifyPaths`.

**F2 — BLOCKING, design, `_lym/plan/03-layout-hub-paths.md` (TestHubPaths).**
`TestHubPaths` plans "one row per moved path function". That duplicates `TestHubScratchAndLogsDir`, which already pins `HubScratchDir`, `HubLogsDir`, `LogArchiveDir` and `AdoptScratchDir`. Card 3 neither moves nor deletes that test, and never names it.
- This breaks `PATTERN-test-speed`, under which a test "pins behavior no other test covers".
- Evidence: `internal/fabricengine/hubscratch_test.go:18-66`.
- Fix: fold its assertions into `TestHubPaths`' rows and delete it in card 3, keeping its "sibling of the stencils `_lym`" assertion.

**F3 — MEDIUM, design, `_lym/plan/00-overview.md` (Decision: the foundation's tests shrink).**
The method covers only part of the set. The overview says "a test of a moved pure name moves to the layout at tier0", but these engine tests of moved pure names appear in no card's Move or Delete list:
- `TestScopedPathspec` (`fabric_test.go:159`), for card 2.
- `TestCommitResult_Committed` (`committed_test.go:16`), for card 10.
- The seed and sync subject rows of `TestSeedCommitSignature` (`seedsignature_test.go:26-27`), for card 4.
- `TestHubScratchAndLogsDir`, already raised as F2.

They stay as tests of layout names inside the engine's test package. Fix: each moving card moves or folds them into its layout table.

**F4 — LOW, consistency, `_lym/plan/04-layout-ref-names.md`.**
`pattern/PATTERN-single-repo-hub.md:11` says the namespace "is spelled once, by fabric's `RecordsNamespaceRoot`, `RecordsNamespaceHeads` and `RecordsNamespaceTags`". `PATTERN-fabric-boundary` defines "fabric" as `internal/fabricengine`, so after card 4 moves the constants this names the wrong home. Card 4 edits only the engine's `doc.go` sentence.
- Evidence: `pattern/PATTERN-single-repo-hub.md:11`.
- Fix: add that background to card 4's Edit list and name `fabriclayout`.

**F5 — LOW, consistency, `_lym/plan/02-layout-package-classify.md`.**
After card 2 removes `structuralCommittedDirs` and `structuralNeverCommittedDirs`, test comments still name them at:
- `internal/fabricengine/junctionnames_test.go:253-254`.
- `internal/fabricengine/structuraldirs_test.go:1`, the file header.
- `internal/fabricengine/structuraldirs_test.go:25,28`.

Card 2 rewrites only the comment of `TestStructuralNameSets_ForAConfigNamingNeitherStructuralDirectory`.

**F6 — LOW, consistency, `_lym/plan/03-layout-hub-paths.md`.**
The `_bolt` row comment of `geometryTokenOwners` says "fabricengine and runproc take the name from it". After card 3 the engine no longer declares `BoltDirName`; `fabriclayout` does. Card 3 rewrites only the `HubPath` and reserved-name comments.
- Evidence: `internal/lymcwd/enforcement_test.go:229`.

**Tool calls**
1. Grep of docs for `fabricengine.&lt;moved name&gt;`, to find doc readers the pre-read lacked. No hits.
2. A second, broader grep of the same docs. No hits.
3. Grep of `pattern/`, `docs` and `contracts` for moved names, the same docs sweep with bare names. This found the PATTERN-single-repo-hub, PATTERN-lym-files, PATTERN-pair-remote and overview lines.
4. Read of the pairteardown enforcement test, the engine entries in `testOnlyCodeAllowed`, and the functions that call `pairSlug` and `isZero`, to check scans and the callers that cards 8 and 10 name.
5. Grep for value-typed `Mutations` fields and direct marshals of the recorder, plus references to the kept tests, to fix card 6's affected set and F2–F3.
6. Grep for per-package import allowlists naming the engine, for allowlists that moved names would change. It found the runproc allowlist, which needs no change. I did not judge the hubreconcile, landingshed, reportengine and mergeresolve seam-enforcement lists: the tool-call budget ran out before I could read them.
