```yaml
lens: code-affected-set
outcome: reported
```

**Task:** the `code-affected-set` lens, run from the manifest context and two searches.

**Per change, the set found:**

1. **`runStateExemptRels` gains the friction directory and the archive glob (`internal/preflight/preflight.go:24-26`).** Covered.
   - The only caller is `fabricengine.Clean(l, runStateExemptRels()...)` at `preflight.go:82`.
   - The only prose describing the exempt set is `internal/preflight/doc.go:25-30`, and the diff updates it to match the code.
   - `internal/preflightshed/preflight.go:179` and `internal/preflightshed/doc.go:11` mention the worktree-clean check only in passing, and the change leaves them correct.
   - `internal/preflight/report.go` declares only the `CheckWorktreeClean` ID, which the change does not touch.
   - The other `fabricengine.Clean` callers pass no exempt list, so their behavior is unchanged: `loomcli/smoke_prebootstrap_integration_test.go` and `fabricengine/cleanreason_integration_test.go`.
   - Nothing in `PATTERN.md` or `pattern/` restates the exempt set.
     - `PATTERN-durable-vs-ephemeral-state` binds the declaration of `_lym` paths to `lymdirs`.
       - The diff names `_lym` only through `loomengine` and `shedrun` accessors.
       - It stays inside the rule.
     - `PATTERN-fabric-git` exempts read-only `status --porcelain`, and the exclusion pathspec is read-only.
     - `PATTERN-cwd-resolution` and `PATTERN-fabric-isolation` are not engaged: `preflight` is an orchestrator, not an engine, and the new `loomengine` import adds no fabric learning.
   - The record, the board entry and the commit message state no `PATTERN` declarations, so there is nothing for "Declared PATTERN changes landed" to check.
   - The commit has no `Pattern-Departure:` trailer, and none is needed.

2. **`LoomFrictionArchivesRelPattern` is new (`internal/loomengine/config.go:361-365`).** Covered for docs and consumers, with one low-severity gap.
   - Its only consumer is `preflight.go:25`.
   - No doc or `doc.go` lists the neighbouring friction accessors, so no listing needs the new one added.
   - `internal/loomengine/friction_test.go` (lines 17, 39-64) pins `LoomFrictionDir`, `LoomFrictionArchivePrefix`, `LoomFrictionDirRel` and `IsLoomFrictionArchive`, but not the new function.
     - Its format tie to `LoomFrictionArchivePrefix` is held only indirectly, through the preflight integration test.
   - The pattern is not equivalent to `IsLoomFrictionArchive`.
     - The glob `friction-*` also matches a path named exactly `friction-`.
     - `IsLoomFrictionArchive` rejects that name, because it requires at least one character after the hyphen.
     - The `readHaltNotes` reader in `internal/reportengine/stops.go:99` uses the predicate and the preflight check uses the glob.
     - The only effect is that a stray `friction-` entry is exempt from the check.
   - The doc comment's description of git's `*` matching across `/` is accurate for a pathspec without glob magic.

3. **`Clean`'s doc now says an entry may be a pathspec glob (`internal/fabricengine/warpclean.go:20`).** Covered.
   - The code path, `exemptPathspec` to `weftCommitPathspec` to `ScopedPathspec` to `:(exclude)<slash path>`, passes glob characters through unchanged.
   - `ScopedPathspec` uses `filepath.Join`, and `exemptPathspec` applies `filepath.ToSlash`, so separators are handled on Windows.
   - `weftCommitPathspec` is shared with commit and `AnchoredPathHasContent` callers, and the diff changes nothing for them.
   - The exemption works for the deleted tracked halt note too, because it is a pathspec exclusion, not an untracked-only filter.

4. **The test is renamed and widened (`internal/preflight/preflight_integration_test.go:415-476`).** Covered.
   - Searches found no other reference to the old name `TestCheckResolved_RunStateUnderLymIsExempt`, so the rename leaves no dangling reference.
   - The test now covers a committed-then-deleted note, an archive note, a new note and a file under `_lym/plan`, which must still refuse.

**Findings:**

- **LOW, consistency.** `internal/loomengine/config.go:361-365`: the new `LoomFrictionArchivesRelPattern` has no test pinning its relation to `LoomFrictionArchivePrefix` and `IsLoomFrictionArchive`.
  - The glob and the predicate disagree on a bare `friction-` name.
  - `friction_test.go:46-48` and `:63` show the sibling accessors are pinned and this one is not.
  - The `code-tests` lens may own the missing-test point; this is the affected-set view of the same gap.

No other member of the affected set is left wrong.

**Tool calls (2 of 6):**
1. A grep across md, go and yaml files for `worktree-clean`, `runStateExempt`, `run-records root` and `weftExempt`. My context held no search of docs and non-test code for descriptions of the exempt set.
2. A grep of md files and `doc.go` files for the friction accessors, to see whether any doc lists them and would need the new function. My context had no such search.
