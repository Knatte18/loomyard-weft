```yaml
lens: code-correctness
outcome: reported
```

Lens code-correctness: I traced every function and branch the diff adds or changes against concrete inputs.

**Per function or branch: the case I ran, and whether it is correct**

- `untrackedWorktreeFiles`, git succeeds with output `"a\x00b\x00"`. The split gives `a`, `b` and a trailing empty string, and the empty string is dropped. Correct.
- `untrackedWorktreeFiles`, git succeeds with empty output. It returns an empty map and true. Correct.
- `untrackedWorktreeFiles`, root outside any repository, so git exits 128. `gitexec.Run` returns a `*GitError`, the function logs at info and returns nil and false. Correct.
- `untrackedWorktreeFiles`, path with odd bytes. `-z` turns off quoting, so the path is kept as raw bytes. Correct.
- `checkStrayFiles`, the listing after the run fails. It returns `result` unchanged. Correct, and the doc says so.
- `checkStrayFiles`, `before` is empty and `after` has `x`. `x` is named. Correct.
- `checkStrayFiles`, a passing run (exit 0, outcome "passed") that leaves one file. Exit becomes 1, outcome "failed", and the line is added to the log, stderr and causes. Correct.
- `checkStrayFiles`, a timed-out run (exit 1, `TimedOut` set). Exit is kept and the line is added. Correct.
- `checkStrayFiles`, a run cancelled by ctx (exit non-zero from the kill). The check still runs and names files that the killed tests' cleanup never removed. The run already failed, so this only adds noise. Not filed.
- `checkStrayFiles` on a `--copies` merged result (`copies` not empty) where every copy passed but a stray file appears. The merged record gets exit 1 and the cause goes on `merged.causes`, but `report` → `addFailedRuns` walks `result.copies` and skips every copy with exit 0. The failure report gets no entry, and the stray cause never reaches it. When some copy did fail, the stray cause is still dropped, because `addFailedRuns` reads `one.causes`, never `merged.causes`. **Incorrect (F1).**
- `strayFilesLine` with 10 files (boundary). No "and N more". Correct.
- `strayFilesLine` with 11 files. Names 10, adds " and 1 more". Correct.
- `runTier`, no invocation. No listing; `runTierCommands` returns a pass. Correct.
- `runTier`, the listing before the run fails. `listed` is false and the run is not checked. Correct.
- `runTier`, refusal. The check is skipped and the refusal is returned unchanged. Correct.
- `runTier` under `retryTier`. Attempt 1 fails on a shortage marker (or crashes with exit 134) and leaves `x.lock`. Attempt 2 calls `runTier` again, whose listing before the run already holds `x.lock`. When attempt 2 passes, the call passes and `x.lock` stays in the worktree unreported. The first attempt's record carries the stray line in its log only. The stray check can also start a rerun: a passing run whose log carries "resource temporarily unavailable" is turned into exit 1 by the check, `failedOnShortage` then reruns it, and the rerun hides the file. **Incorrect (F2).**
- `runTierCopies` after the rename to `runTierCommands`. Each copy runs once with no listing of its own, and the one listing wraps the whole copies run. Correct.

**Findings**

- **F1 — MEDIUM, design.** Location: `internal/gatecli/strayfiles.go:52-60`, with the consumer at `internal/gatecli/copies.go:884-896` (`addFailedRuns`).
  - Summary: on a `--copies` run, the stray-file failure is set on the merged result only, so the failure report drops it.
  - When every copy passed, the call exits 1 with no failed run listed.
  - When a copy failed, the stray cause is missing from that copy's report entry.
  - Evidence: `copies.go:889-891` skips copies with `ExitCode == 0` and uses `one.causes`; `strayfiles.go:57` appends to `result.causes` on the merged result; `run.go:722-723` calls `addFailedRuns`.
- **F2 — MEDIUM, design.** Location: `internal/gatecli/tierrun.go:71-87` under `retryTier` (`tierrun.go:344-378`, called from `run.go:629`).
  - Summary: a rerun lists the first attempt's stray files as already present, so a file left by a first attempt that failed on a shortage or crashed escapes the check when the rerun passes.
  - The check's own exit 1 can also cause that rerun when the log carries a shortage marker.
  - Evidence: `tierrun.go:75` lists the files again on every `runTier` call; `tierrun.go:345-378` returns only the rerun's result; `failedOnShortage` (`tierrun.go:303-307`) reads `ExitCode != 0` and the log after the stray check has set them.

Tool calls: none.
