```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the F1–F12 change set agrees with its surroundings (fix-consistency).

1. **LOW**: `_lym/landing/summary.md` was not updated with the change set.
   - Its Tests section lists no `newbinary_test.go` and no `TestSumThrottleCounts`.
   - It describes `probe_test.go` as one successful run "pinning … the skipped lym starts". The file now also has a failed-run cleanup test and a no-`-dir` run.
   - "What it measures" records a `skipped` entry only for the lym starts, but `probe.go` now also adds `newbinary, fileops: no -dir folder given`.
   - It says folders are removed "on failure too" and says nothing of interrupt or SIGKILL, while the `main.go` package doc now states both.
   - It does not mention the pinned build environment or the dropped `GIT_*` variables.
   - The file is under `_lym`, so the parent decides whether it is in the write surface.

2. **LOW**: `gitload.go` `measureGitWorkload` drops the caller's `GIT_*` variables, but its twin timed git measurement does not.
   - `probe.go` runs `measureStarts(ctx, "git_version", []string{gitBinary, "--version"}, …)`, and `starts.go` leaves `cmd.Env` nil.
   - A caller's `GIT_TRACE2_EVENT` is therefore still written on every timed `start/git_version` start. That is the distortion the new comment names ("trace the timed rounds").
   - No stated doc claim is broken, since the package doc speaks only of the workload.

3. **LOW**: the new `linkProbeProgram` doc sentence in `newbinary.go` reads wrongly: "the build uses no workspace, go env file, GOFLAGS or cgo, the local toolchain, and the default experiments and amd64 level."
   - The "no" carries across the list, so it can be read as "no … local toolchain" and "no … default experiments".
   - The code sets `GOTOOLCHAIN=local`, `GOEXPERIMENT=` and `GOAMD64=`, so these are used. A second verb or sentence would fix it.

These agree and need no change: the `-dir` wording across the package doc, flag usage, `usageText`, error and tests; the `probeFolderPrefix`, `trapsPath` and `lymStartName` sites; the updated test file headers; the serial-test comment; and every caller of the changed signatures.

Tool calls:
1. `cat _lym/landing/summary.md`: the parent never read it, and it describes the changed behaviour and tests.
