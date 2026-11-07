```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fixer's changes agree with the sibling decisions, criteria and cross-references in the decision record.

1. **LOW**, the change at D1 "The relative-`gitdir` parity test needs git 2.48 or later to write one, and skips below it, naming the version."
   Summary: the twin acceptance criterion was left as it was. "D1: a linked worktree whose `gitdir` holds a relative path resolves to the same path git reports" says nothing about skipping below git 2.48.
   Evidence: the new D1 text sets a skip, but the Acceptance criteria line for the relative `gitdir` still has no condition. A reader of the criteria alone would expect the test to run on the 2.43.0 floor.

2. **LOW**, the change at D2's TB bullet, which now refuses "`Cleanup`, `Setenv`, `TempDir`, `Chdir` and `ArtifactDir`".
   Summary: the next line was not updated. "The helpers a shape build uses today, `AddPair`, `AddPairWith` and the `gitkit` commit helpers, register none of them" fits the three registering methods, but `Chdir` and `ArtifactDir` are not registrations. "Call none of them" would cover all five.
   Evidence: the D2 bullet's continuation line, which this change left as it was.

3. **LOW**, the change at D1's Bound (both callers now fall back to `git worktree list --porcelain` on a reftable repository).
   Summary: the twin in Constraints still describes the webster read as `dotgit` only. `PATTERN-fabric-boundary` says "`internal/websterengine` reads its own worktree's siblings through `dotgit`, as it reads them through git today", with no mention of its porcelain fallback.
   Evidence: the Constraints bullet `PATTERN-fabric-boundary`. The `PATTERN-gitrepo-client-boundary` bullet ("the porcelain fallback stays on `gitexec`") is general enough to still hold.

Tool calls:
- One grep of `_lym/discussion/support-log.md` and `relevant.md` for `TestNoticeSite`, 2.43, five keys, "behind its upstream", `TempDir` and reftable. My context did not hold those files, and I needed to see whether a sibling record cites text this change rewrote. The only hits were history entries (Q2, Q4, walkthrough 1), and none of them contradicts the changes.
