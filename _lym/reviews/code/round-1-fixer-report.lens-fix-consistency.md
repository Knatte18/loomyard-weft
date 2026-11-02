```yaml
lens: fix-consistency
outcome: reported
```

Task: check the change set's agreement with its surroundings (fix-consistency lens).

Findings:

1. **LOW** — `docs/benchmarks/test-suite-timing.md:812`, "ran alone after the shm run's start".
   - The sentence now contradicts itself and the table. It says the default run "waited for the shm run's slot", so it started after the shm run's slot was free. It also says the default run "ran alone after the shm run's start", which would put it beside the shm run.
   - Table row 2 shows the default run with a gate wall of 350.2 s against 454.9 s for the shm run, and both rows share one load-at-start value (46.05 37.62 27.82).
   - A reader cannot tell from the text or the table whether the default run overlapped the shm run, so "alone" is unsupported. The replaced "14:24 to 14:30" window was at least a definite claim.
   - Fix direction: state only what the table shows, or give the window in a checkable form. Do not keep an unverifiable "ran alone" claim.

2. **No finding** — the F3 rename and the F4 row.
   - `grep` finds no leftover `runNameDigits`, `r0a1b2c` or "million names" outside the earlier report.
   - `docs/benchmarks/test-suite-timing.md:736` and `:767` (the earlier report) were rejected by the review.
   - The fixer left `internal/lymtemp/doc.go` untouched; the previous commit, `HEAD~4..HEAD~3`, changed it (3 lines).

3. **No finding** — the `revision: e1770b2a` line (F1).
   - It matches the commit that shortened the run directory name.
   - `9bb10058` after it only reflowed comments in `internal/lymtemp/doc.go` and `internal/lymtemp/run.go`.

Tool calls made:
- A `grep` across `*.go`, `*.md` and `*.yaml` for the retired names. My context did not show the repository-wide twins.
- A `git diff HEAD~3 HEAD` on the benchmark file, plus `--stat` for `HEAD~4..HEAD~3`. My context did not show the committed form of the doc edits or the earlier commit's scope.
