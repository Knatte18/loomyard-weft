```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. List, per change, of what the diff's changes force outside the lines it touches.

1. **`seed` / `Spawn` launch path and `windowDir` for a beam slug.**
   - The only production callers of `ideengine.Spawn`, `Seed` and `OpenTask` are `internal/idecli/cli.go:77`, `:106` and `:111`. The reed `code` button reaches `Spawn` through `lym ide spawn {slug}` (`internal/reedengine/template_posix.yaml:30,37` and the Windows template).
   - No caller reads `launchPath` or `windowDir`. Both are private to the `seedResult` in `spawn.go`.
   - Covered.
   - I found one stale caller outside Go: `fabricengine/launchers.go:140` generates a launcher script running `ide spawn <slug>`. It passes only the slug, so it picks up the new behavior unchanged and needs no edit.

2. **`lym ide spawn <beam>` documentation.**
   - `docs/overview.md:399-404` is updated, and `:411` already described the beam file.
   - The `spawn.go` file header and doc comments and the `idecli/cli.go` header are updated.
   - `fabricengine/doc.go:1188-1191` and `pattern/PATTERN-cwd-resolution.md:46` describe the files, not the spawn behavior, so they stay accurate.
   - Covered.

3. **`lym ide seed <beam>` now regenerates the workspace files.**
   - Documented in the `Seed` doc comment and the `cli.go` header.
   - `docs/overview.md:710` describes Seed-IDE for a task worktree. A task slug is unchanged, so it stays accurate.
   - `internal/idecli/cli_test.go` and the other `Spawn` tests in `spawn_clean_integration_test.go` use task or prime slugs. None asserts an old beam result.
   - Covered.

4. **`isBeam` reads `ReadOriginFor`.**
   - The doc comment's claim that the record is the mark regeneration writes a beam's file by matches `hubworkspace.go:351`, `pair.beam = origin.Beam`.
   - `ReadOriginFor` returns `found=false` with no error for an absent record. Reading it takes the records lock and creates nothing.
   - `ReadOriginFor` errors when the weft worktree lacks its `.weft` lock directory. That reaches the bare folder through the Warn path, as the `Spawn` doc comment says.
   - Covered.

5. **PATTERN entries.**
   - `PATTERN-fabric-isolation` is not touched. `ideengine` imports `fabricengine`, which is the allowed direction, and fabric learns nothing new.
   - `PATTERN-cwd-resolution` is not touched. The diff passes slugs through `lymcwd.NamedWorktree` and writes no workspace file into a worktree.
   - The commit message carries no `Pattern-Departure:` trailer, and none is needed.
   - No declared PATTERN changes were listed for this round, and the diff touches no `PATTERN.md` or `pattern/` file. Nothing raised.
   - Covered.

6. **Test fixtures.**
   - The new scenario step is added in `spawn_scenario_integration_test.go`. It uses the slug `beam-workspace`, and `MarkBeam` runs inside the step, so later steps do not see the mark.
   - Covered.
   - Duplication against the prime and task checks is the code-tests lens's call, not mine.

7. **Commit message.** It matches the diff. The `Co-Authored-By: Claude Opus 5.5` trailer differs from the attribution line the reminder gave, but that is not a diff defect.

**Findings:** none.

**Tool calls (2 Bash, plus a second search folded into the third call):**
- A grep for `ide spawn`, `bare folder`, `code-workspace` and `BeamWorkspacePath` across md, yaml and go files. I needed it to find stale docs and callers that the manifest lacked.
- A grep of tests under `idecli` and `ideengine` for `Spawn(` and `Seed(`. I needed it to find other tests that assert a beam result.
- A grep of `fabricengine/doc.go`, `vscode` and the cwd-resolution pattern file for claims about prime-spawn regeneration, plus a read of `spawn.go:185-200`. I needed it to confirm the `primeResolved` degrade path.
