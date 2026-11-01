```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. Per behaviour, the test that pins it, or none.

**Pinned**
- D1 unknown keys survive edits, an absent file holds no entry, order ties break by slug, set refusals, partial set, malformed file: `store_test.go` `TestStore` (`UnknownKeysSurviveEdits`, `AbsentFileHoldsNoEntry`, `OrderedBreaksTiesBySlug`, `SetRefusals`, `PartialSetChangesOnlyGivenField`, `MalformedFileFailsLoadWithWayForward`).
- D2 verbs and refusals: `cli_integration_test.go` `TestHeddleCLI` (`EntryVerbs`, `RefusalsSaveAndSyncNothing`, `SyncFailureKeepsTheSave`, `WriteHoldsTheBoltLockAndCommitsNothing`, `HubResolutionRefusals`, `RenderChoosesAnExistingWorktree`, `RenderSeedsAndWritesThePrime`, `ShowWritesNothing`).
- D3 render rules in `render_test.go` `TestRender`:
  - join order and `@AGENTS.md` imports: `ClaudeFileJoinsEntriesInOrder`, `ClaudeFileImportsAgentsFiles`
  - codex root `AGENTS.md`: `CodexFileOpensWithRootAgents`
  - stale marked render removed off the exclude list: `StaleMarkedRenderIsRemovedEvenOffExcluded`
  - unmarked file left alone: `UnmarkedFileIsLeftAlone`
  - off-exclude refused: `OffExcludedTargetIsNotWritten`
  - blocked `.heddle`: `BlockedTempDirWritesNoFile`
  - leftover temp file overwritten: `LeftoverTempFileIsOverwritten`
  - Bolt lock held: `HeldBoltLockFailsTheRender`, `ReadWaitsForTheBoltLock`
  - two renders of one worktree: `RenderLockSerializesRenders`
  - no untracked file: `render_integration_test.go` `TestRender_LeavesNoUntrackedFile`
- D4 prompt seam: `prepare_test.go` `TestPrepare_AppendsStandingPassages` (two passages, no reader, outside a hub, reader error).
- D5 exclude seeding: `warpexclude_integration_test.go` (seeded once and warp only, kept by remove, rewire, `AddLocal`, unparseable config).
- D6 events, in `worktreeseam_integration_test.go` and `seam_test.go`:
  - no seam installed, and made after clone and add: first two rows
  - failing seam on add and on pull (result, log, output, hub-pull line)
  - fast-forward, reset, Bolt and weft fire nothing, parent-pair fast-forward, `ReconcileOrigin` code branch
  - heddle's `TookIn` path selection: `TestSeam`
  - pull end to end: `TestSeam_PullTakeIn`
  - root pre-run install: `prerunseams_test.go` `TestPersistentPreRunE_InstallsSeams`
- D7 registry in `boltreplayers_integration_test.go` `TestBoltReplayers`:
  - unclaimed cherry-pick
  - claimed commit with no run ledger fails and leaves the tip
  - ended-set and owner read with no ledger fail
  - non-ledger loss tagged under the replayer
  - ledger loss with no entry slug
  - `replay_integration_test.go` rows: heddle on both sides, board and heddle in one commit, ended board plus heddle keeps heddle and tags the original, heddle field clash
  - no board name in fabric's replay code: `boltreplay_test.go` `TestBoltReplay_NamesNoBoardFile`
- D7/D8 heddle merge: `replayer_test.go` `TestReplayer`.
- D8 shared merge: `entrymerge_test.go` `TestTable`. The board's replay cases (`boardengine/replay_test.go`) cover the origin-moved-to-a-third-value loss.
- D9: `lymcwd/enforcement_test.go` `TestEnforcement_BoltDirNameDeclaredOnce` and the narrowed owner row.

**Findings**

1. **MEDIUM, design: no test for a resume that keeps the append-system-prompt file.**
   - Location: `internal/shuttleengine/claudeengine/prepare_test.go:447-500`; behaviour at `claudeengine.go:146-148`.
   - D4 and its acceptance line require that a resume through the launch's own resume line keeps the file, passages included.
   - `TestPrepare_AppendsStandingPassages` only runs a fresh `Prepare`.
   - The existing `TestPrepare_ResumeSessionID` (line 344) was not changed by the diff, and I did not read it. Nothing visible asserts the passages survive or are not re-read.
   - Not verified: whether that test pins this.

2. **LOW, design: the `Prepare` seam test has no case where the reader returns zero passages.**
   - Location: `prepare_test.go:447`.
   - A reader that succeeds with no entries should leave lym's sentences with no trailing blank-line separator.
   - `no_reader` and `outside_hub` do not reach the join logic.
   - Not verified: whether the join code special-cases an empty list. I did not read it.

3. **LOW, design: no test that `claude-prompt` entries and `codex-prompt` entries stay separate through the reader.**
   - Location: `internal/heddleengine/render.go:285` (`Passages`, `engine+"-prompt"`).
   - `ShowAndPassagesWriteNothing` is the only row I saw for `Passages`.
   - I did not see a row feeding a `codex-prompt`-only entry to the `claude` reader and expecting none. Weak evidence; not verified beyond the subtest name.

I found no test economy violations from the names I saw. None of the added tests is a per-helper duplicate of a public-surface test.

**Tool calls**
1. Grep of test function and subtest names across the new test files: my context had only the file names.
2. Read of the diff of `replay_integration_test.go`, the passage test rows and the lymcwd enforcement diff: my context lacked their contents.
3. Read of `prepare_test.go` lines 447-530, plus a grep of `claudeengine.go` for resume handling: to check the D4 resume behaviour. The `prepare.go` path in that grep did not exist.
