```yaml
lens: code-tests
outcome: reported
```

**Per-behaviour list (code-tests lens)**

1. **A new untracked friction note is exempt.**
   - Pinned by `TestCheckResolved_LymOwnStateIsExempt`, the `Darn-Write.md` write at `internal/preflight/preflight_integration_test.go:440`, at root and `backend` anchors.
   - The unscoped `fabricengine.Clean` assertion at `:458` proves the note is visible first. Dropping the `LoomFrictionDirRel()` entry makes `assertCheckSet` at `:466` fail.

2. **A committed halt note deleted from the friction directory is exempt, the issue's main case.**
   - Pinned at `:432-438` plus `:466`.
   - The fixture's `PrimeRecords()` is `fabricengine.RecordsWorktree(h.Location)` (`internal/hubforge/hub.go:223`), the same worktree `Clean` probes, so the commit and the deletion are seen on the same side.
   - This row also fails if the `:(exclude)` pathspec stops hiding tracked deletions.

3. **An untracked file in a timestamped archive sibling is exempt.**
   - Pinned by the write at `:439` with the `20261103-120000` suffix.
   - Dropping `LoomFrictionArchivesRelPattern()` from `runStateExemptRels` fails `:466`, because the friction directory entry alone does not cover the sibling.
   - The subpath anchor `backend` exercises the glob joined under `AnchorRel`.

4. **Everything else under `_lym` still refuses.** Pinned by the `plan/plan.md` write at `:468` and the `CheckWorktreeClean` assertion at `:473`.

5. **The run-records root stays exempt.** Pinned by the `runState` list at `:443-453`, which was unchanged and still passes.

6. **`LoomFrictionArchivesRelPattern()`, new exported function.**
   - Pinned only indirectly, through the integration test (`:439`).
   - No direct row in `internal/loomengine/friction_test.go` beside the `LoomFrictionDirRel` and `LoomFrictionArchivePrefix` rows, and no row that the pattern agrees with `IsLoomFrictionArchive`.
   - A drift that over-matches or under-matches a non-archive sibling, such as `friction` itself or a `frictionx` lookalike, fails no test.
   - Finding: LOW, class `design`.
   - Location: `internal/loomengine/config.go:363`, `internal/loomengine/friction_test.go:46-64`.
   - Fix is one row beside the existing `IsLoomFrictionArchive` table, not a new test function.
   - The integration test covers the happy path, so this is a coverage gap on the over-match boundary only.

7. **Windows separators in the glob pathspec.** None. The code applies `filepath.ToSlash` (`warpclean.go:59`), and the test has no Windows-only row. I judge this too narrow for a finding, since the fixture runs on the host OS.

8. **`Clean`'s doc change (pathspec globs allowed).** Doc only, exempt. The glob path is exercised by item 3.

9. **Test economy.**
   - Each new fixture line drives a distinct exempt path: the committed-then-deleted note (item 2), the archive entry (item 3) and the new note (item 1).
   - They are rows inside the one existing test, which was renamed. No redundant test function was added.
   - Nothing for the `PATTERN-test-speed` redundancy rule to flag.

No BLOCKING or MEDIUM findings from this lens.

Tool calls (1 of 6):
1. `grep` of `setupAnchoredFixture` and `Hub.PrimeRecords`: my context lacked the fixture and the `PrimeRecords` definition, which I needed to confirm the commit and `Clean` use the same records worktree.
