```yaml
lens: code-correctness
outcome: reported
```

**Lens: code-correctness.** Cases run, per function or branch:

- `AnchoredPathHasContent`, tracked note deleted on disk (first seed's clear): `ls-files --cached` still lists it, so it returns true. The pathspec then stages the deletion. Correct.
- `AnchoredPathHasContent`, untracked-only or empty directory: nothing is listed, so it returns false and the pathspec is omitted. Git never sees a non-matching pathspec. Correct.
- `AnchoredPathHasContent`, only ignored files in the directory: `--exclude-standard` drops them, so it returns false. Correct.
- `AnchoredPathHasContent`, `weftCommitPathspec(...)[0]`: `ScopedPathspec` returns one entry per input (`fabric.go:224`), so index 0 always exists. With `AnchorRel` ".", `filepath.Join` gives the bare relative path. Correct.
- `AnchoredPathHasContent`, `git ls-files` failure: the error is wrapped and returned. Correct.
- `bootstrapCommitPaths`: false returns the three base paths. True appends the friction directory, and the slice is fresh each call. Correct.
- Step 3 probe, `c.frictionDir == ""` (Tier 2 off): the probe is skipped and the friction directory is never added. Correct.
- Step 3 probe, genuine first seed over tracked notes: the clear deletes them and `EnsureDir` recreates the directory empty. The probe is true via the cached entries, the deletion is committed with the seed, and Preflight sees a clean tree. Correct.
- Step 3 probe error path: it returns `bootstrapStageCommit` with empty strings. Correct.
- Step 3 probe with `FABRIC_SKIP_GIT=1`: not correct, see F1.
- Step 3 probe on an `ErrSeedExists` resume with friction content: not correct, see F2.

### Findings

**F1 — LOW, design**
- Location: `internal/loomcli/sharedbootstrap.go:164-170`
- Summary: The probe runs `git ls-files` in the weft worktree even when `FABRIC_SKIP_GIT=1`, though `CommitAnchoredPaths` is a documented no-op under `SkipGit`.
- Why: In an environment with no usable weft git, the probe can now fail the bootstrap with `bootstrapStageCommit`. The commit it guards would have been skipped anyway.
- Evidence: `fabric.go:107-111` shows `EnvSyncOptions` reads `FABRIC_SKIP_GIT`. `commitweftpaths.go:58-60` shows the early return under `SkipGit`. `commitweftpaths.go:108` shows the probe has no `SkipGit` check.
- Fix: skip the probe when `fabricengine.EnvSyncOptions().SkipGit` is set, or take the `SyncOptions` into `AnchoredPathHasContent`.

**F2 — LOW, consistency**
- Location: `internal/loomcli/sharedbootstrap.go:160-171`, with `ensureFrictionDirAfterSeed` at `:240-244`
- Summary: The probe runs on every invocation, so an `ErrSeedExists` resume with tracked or untracked friction notes now commits them under the message "loom: seed session bootstrap for <slug>".
- Why: The resume branch's own doc says a resume's notes are left untouched, and now they are staged and committed. This is harmless to the Preflight outcome and consistent with the stated self-heal, but the commit message mislabels the content. It is also a behaviour change beyond the first-seed clear.
- Fix: either accept it and say so in the step 3 and `ensureFrictionDirAfterSeed` docs, or gate `withFriction` on a first-seed signal. The signal would be `seedErr == nil`, but that loses the crash self-heal the comment claims.

I found no wrong result, swallowed error or inconsistent failure state beyond F1 and F2.

### Tool calls
1. `grep` for `EnvSyncOptions` and `ScopedPathspec` in `internal/fabricengine`: my context lacked their definitions. I needed them for the `SkipGit` case and the `[0]` index.
