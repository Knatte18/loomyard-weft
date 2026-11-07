```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. For each decision, I traced the existing code paths that write or read the state it changes.

**Paths traced, per decision**

- **D1, `List` callers that pass `AnchorPath()`, which is a subdirectory at the `backend` anchor** (`hubgeom/parentof.go:36`, `worktreelist.go:114`, `addparent.go:34`, `hubpull.go:193`, `reedcli/list.go:43` and others): **breaks unless the reader walks up.** `dotgit.Read` needs `.git` directly in the directory it is given, and the record does not say how the reader finds the repository from a subdirectory.
- **D1, `websterengine.otherWorktrees`** canonicalises each path, a prunable one included, as today: survives.
- **D1, `matchParentBranch` and `PrimeName`** read `Prunable` and `Main`, and use `FromSlash` and `Base`: survive, given the parity D1 promises.
- **D1, `prune.go` and `cleanup.go`** read `List`, then call git: survive.
- **D2, `markFixtureBuild`**, the process-wide `FixtureGitEnv` mark shared by builds running at once: survives. No production code reads it outside `gitkit` and `hubforge`.
- **D2, `hubAt` and `RegisterTeardown` on a registered template's copy**: survives. Links are relocated, and the teardown walk is slug-free.
- **D2, process-global test seams set by the template's first caller**:
  - `SetWeftHeadSHAForTest`, `SetAddBeforeWeftReplaceHookForTest` and `SetAddFailAfterWarpHookForTest` (`export_test.go:53`, `:631`, `:641`) and `hubreconcile.SetSpawnBoardSyncForTest` are active while the build's `AddPair` and commits run.
  - **Breaks**: whatever those seams do to the build ends up in every later copy.
- **D3, `TestStoreHousekeeping` loose-object checks after a packed template**: survives. The test writes its own loose objects and samples `objects/17` (`housekeeping_integration_test.go:70-77`, `:157`, `:201`).
- **D3, `trackingReflog` readers** (`pull.go:664`, `originreconcile.go:604`, `pushclassified.go:495`): survive. The fold stays on `git push`, as today.
- **D3, the fold `push --force origin HEAD` after a builder that commits straight into the records bare** (`endedHub`, then `originBoard`): **breaks**. See finding 1.
- **D3, relocation of packs and `packed-refs`**: survives. Packs are binary and skipped, `packed-refs` and `objects/info/packs` hold no template path, and `findRootReference` still checks the result.
- **D3, `SharedHub` cleanup fingerprint**: survives. The pack runs before the build-time fingerprint.
- **D4, `gitkit.rewriteOriginURLInConfig`** (`gitkit.go:222`), which rewrites `.git/config` as text: survives the one Go-written block.
- **D4, production `disableStoreAutoGC`** writes hub stores, not test initialisers: no overlap.
- **D5, `InstallPostCheckoutHook`** returns early on a hook that holds the sentinel (`hook.go:81-83`): survives. The record's claim that old hubs keep their script holds.
- **D5, `newRecordsFixture` on a copy, and `readEndedSet`**: survives. The copy's board reflog shape is the same as today's pair-shape copies.
- **D7, loomcli `beamHub`** writes `board.json` and the shortname file uncommitted: survives. The fold commits only the binding, so copies are dirty exactly as today's fixture leaves them. D2 does not require a clean board.
- **D7, battencli `beamHub`** uses `SkipPush` pairs: survives. `Add` with `Beam` writes the pair's origin record, not the board (`add.go:48`, `:653`).
- **D7, `newOriginHub` and `originPair`**: survive. They push the prime records branch and set a pair upstream, which are not the board.
- **D7, `newStaleHub`**: survives. Its commits land on records worktrees, not the board.

**Findings**

1. **MEDIUM · design · D2 board rule, D3 fold, D7 `endedHub`**
   - **Summary:** D3's force-push of the folded board tip overwrites the commit a builder made straight in the records bare. D2's guard does not catch it, so every copy of an `endedHub` template loses origin's done board.
   - **How it happens:**
     - `endedHub` runs `publishBoard`, which commits `board.json` on the board and pushes.
     - It then runs `originBoard`, which commits a newer `board.json` straight into `hub.RecordsBare` on the same branch.
     - The local board's HEAD still equals its tracking ref, so D2's "board's HEAD equal to its upstream tip" passes when read as `@{u}`.
     - D3's fold then runs `push --force origin HEAD`, and D3's own acceptance criterion requires "the copy's records bare's branch tip is the board's HEAD". Origin's done commit is dropped.
   - **The gap:** D7 lists `endedHub` among the moves. The record never says whether "upstream tip" means the local tracking ref or the bare's branch, and it never says that a builder writing the remote directly has to stay in the test.
   - **Evidence:**
     - `internal/fabricengine/originreconcileended_integration_test.go:58-71` and `:87-100`
     - `internal/hubforge/template.go:187`
     - `_lym/discussion/decision-record.md:92`, `:109`, `:158`, `:160`, `:242`

2. **MEDIUM · design · D2 Bound**
   - **Summary:** The build runs inside whichever test copies the template first, under every process-global seam that test has installed, and the result spreads to every later copy. The Bound covers only process-wide calls the build itself makes.
   - **Evidence:**
     - fabricengine's package-level test seams that `Add` reads: `SetWeftHeadSHAForTest`, `SetAddBeforeWeftReplaceHookForTest`, `SetAddFailAfterWarpHookForTest` (`internal/fabricengine/export_test.go:53`, `:631`, `:641`)
     - `hubreconcile.SetSpawnBoardSyncForTest`, swapped by a non-parallel test (`internal/hubreconcile/ensure_integration_test.go:477`)
     - `_lym/discussion/decision-record.md:85-99`, which states no-per-call-data and the Bound, and is silent on what the first caller has installed.

3. **LOW · design · D1**
   - **Summary:** The reader is said to read "from the common dir", but many `List` callers pass `AnchorPath()`, a subdirectory at a non-`.` anchor, and the record does not say the reader walks up to the repository. `git worktree list` accepts any subdirectory; `dotgit.Read` does not.
   - **Evidence:**
     - `internal/dotgit/dotgit.go:32-36`, where `Read` needs `.git` in the directory itself
     - `internal/hubgeom/parentof.go:36`, `internal/fabricengine/worktreelist.go:114`, `internal/reedcli/list.go:43`
     - `_lym/discussion/decision-record.md:33-35`

**Tool calls**

1. `housekeeping.go`'s gc logic, `hook.go`'s install path and `reflog.go`'s functions: the manifest held neither the housekeeping probe nor the hook's idempotence check, which D3 and D5 depend on.
2. `housekeeping.go`'s gc body and `TestStoreHousekeeping`'s loose-object assertions: to check D3's packed template against that test.
3. Callers of `trackingReflog`, readers of `FixtureGitEnv` and the ended-set readers: to check D3's fold and D2's builds running at once against the code that reads that state.
4. `boardkit.InstallReplay`, and `endedHub`'s `publishBoard` and `originBoard`: to see what the `endedHub` builder writes to origin.
5. The process-global `Set*ForTest` seams in `hubreconcile` and `fabricengine`: to see what a first caller can have installed while a build runs.
