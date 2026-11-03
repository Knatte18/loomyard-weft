```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

I ran each changed branch by hand on one input. All of them hold.

- **`seedAndCommitBootstrap` step 3:**
  - Under `FABRIC_SKIP_GIT=1`, the probe is skipped and `withFriction` stays false.
  - `CommitAnchoredPaths` then returns early on the same `syncOptions`, so no git runs.
  - With `frictionDir` empty, the probe is also skipped, as before.
  - If `AnchoredPathHasContent` errors, the bootstrap returns at `bootstrapStageCommit`, as before.
  - `err` is declared earlier in the function and `go vet` passed, so the `=` assignment compiles.
- **`TestAnchoredPathHasContent`:** each row fails if the probe's `ls-files --cached --others --exclude-standard` flags are changed.
  - `TrackedFileDeletedOnDisk` expects true. It fails without `--cached`, because the deleted file is not listed as untracked.
  - `UntrackedFile` expects true. It fails without `--others`.
  - `IgnoredOnlyDirectory` expects false. It fails without `--exclude-standard`.
  - `EmptyDirectory` expects false, since git lists no entry for an empty directory.
  - Tier 1 passed in the parent's gate run.
- **Resume assertion in `TestSmokeStepBootstrapWiring`:** `LsFiles` runs `git ls-files -- <spec>`, which lists cached files.
  - If step 3 skipped the probe on resume, the note would stay untracked and `len(tracked) != 1` would fail.
  - `filepath.Join(".", ...)` cleans to a plain relative path, so the spec matches.

Open risk (LOW, unverified):
- **Possible false pass:** I did not check whether another part of `step` (for example a later records commit) also commits the planted note. If one does, the assertion would pass without step 3's friction commit, so it may not pin the fix.
- **Tier 2 run not confirmed:** the parent's tier 2 run reported 35 passed and 248 skipped, and the log summary doesn't show whether this smoke test was among them.

Tool calls made:
1. Read `internal/gitkit/query.go:74-85`, to confirm `LsFiles` passes its arguments straight to `git ls-files`. My context lacked that helper's body.
