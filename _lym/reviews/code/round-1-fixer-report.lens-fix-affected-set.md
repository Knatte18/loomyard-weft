```yaml
lens: fix-affected-set
outcome: reported
```

Task: checked that the fixes cover every member of each set-shaped finding (F1, F2, F6, F7, F8, F9, F12).

Findings left unchecked: none.

1. **LOW**: `tools/machinecost/probe.go:61`, the F12 text `"newbinary, fileops: no -dir folder given"`.
   - The fix restates two measurement group names that were written once each in production code, at `newbinary.go:64` (`"newbinary/"`) and `fileops.go:73` (`"fileops/"`).
   - That is the repeated literal F1 raised under `PATTERN-code-conventions` ("one named constant, declared once and referenced everywhere").
   - `"start/"` is likewise written at `probe.go` (the lym Skipped text, as F1 proposed) and at `starts.go:27`.

2. **LOW**: F1's set was covered only at the sites the review named.
   - `"probe.txt"` is written twice, at `gitload.go:128` (the write) and `gitload.go:131` (`git add`). A rename at one site breaks the workload.
   - The default report name `machinecost-<host>-<UTC time>.json` is restated in the flag usage (`main.go:118`) beside the format that builds it (`main.go:157`).
   - Both predate the fix, but they fall under the same rule as F1, which was BLOCKING.

The other sets are covered:
- **F1 named sites:** the folder prefix, the traps path and the lym start name are each declared once in production code. The literals left in tests are expected values.
- **F2:** every measuring loop stops on a cancelled context, through `CommandContext` or the `ctx.Err()` check in `fileops.go`. The only plain `exec.Command` calls left are the two short version queries in `readMachine`.
- **F6 and F7:** `go build` is the only go command that makes a work folder, and `go env` is pinned to the local toolchain.
- **F8:** the whole workload runs on the filtered environment. The `git --version` calls still inherit `GIT_*`, which is outside the finding's premise.
- **F9:** the package doc, usage text, flag help and error message all say `name=path`, and so does `_lym/landing/summary.md:28`.
- **F12:** the no-`-dir` case is the only silent skip.

Tool calls (3 of 6):
- grep for repeated name and path literals in `tools/machinecost/*.go`: my context had no single view of every site after the edits.
- grep for process starts, loops and folder creation in the same files: needed to rerun the F2, F6 and F8 searches over the changed code.
- grep for `-dir` form wording and `machinecost` mentions in the tool, docs and `_lym/landing`: my context had not shown the docs or the landing summary.
