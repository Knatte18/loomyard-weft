```yaml
lens: fix-proposal
outcome: reported
```

Findings, from the proposals the fixer carried out as proposed:

1. **LOW** — F1, `cutLine` in `failurereport.go`, which `renderFailureReport` caps with `capLine(cutLine(cut))`.
   - **Summary:** the cut line shares the 300-byte per-line cap, so on a call with many cut runs the hint that tells the reader where cut lint findings can be read is the part that gets truncated.
   - **Evidence:** the hint is the tail of the line, after one `label: n` count per cut run. `capLine` cuts at `failureReportLineBytes` (300).
     - The log hint is about 60 bytes.
     - Each lint hint is about 78 bytes.
     - Take a `vet tier1`, `guard tier1` and `tier0` flood, followed by `lint comments` and `lint pattern`. The line passes 300 bytes, and the `lym loom lint-pattern` hint ends in `...`.
     - The lint findings exist nowhere else, which is the premise of F1. In this case the proposal's cut line fails to say how to recover them. The test covers only one lint.
   - **Possible change:** exempt the cut line from `capLine`, or put the hints before the counts.

2. **LOW** — F6, `runRelativeGoFile` and `qualify`.
   - **Summary:** the regex accepts only `/` separators and `./` prefixes.
   - **Evidence:** the repo has `runtime.GOOS == "windows"` branches in its tests and in `build.go` (`verifyrun.ShellFor`). On Windows the go command prints compile errors with backslashes, as `.\q.go:3:2`. Those lines match neither new regex, so the worktree-relative promise still fails there.
   - The proposal named a path relative to the run directory without saying which separator. The unit rows cover only `/`.

3. **LOW** — F6, the qualify rule keyed on `failure.test == ""`.
   - **Summary:** the rule treats every slash-bearing path in a no-test failure as run-directory-relative.
   - **Evidence:** a panic or a log line outside any test can print a package-relative path with a directory, such as `testdata/x.go:3`. The new branch prefixes it with `moduleDir`, which would be wrong for that line. The old code left such lines as printed. This is a heuristic with no row pinning the boundary, only the `/abs/z.go` case.

4. **No finding** — F2, F3, F4, F5, F7, F8, F9 and F10.
   - **F2/F3:** the new stand-in line is printed only for `sharedbuild`. The other `helperBuildExitEnv` user (line 1437) goes through the prebuild `build`, so it is not affected.
   - **F5:** the only readers of `tierResult.unparsed` are `failurereport.go` and `tierrun.go`, so splitting out `causes` breaks no other consumer.
   - **F9/F10:** the grep for the old wording ("every card gate runs", "runs after lint-comments", "after its tests --") finds no remaining occurrence in `.go`, `.md` or `.yaml` files outside `_lym` and `.lym`.

Tool calls:
1. `grep` for the stale card-gate lint wording across the repo. My context lacked a repo-wide search for twins of the F9 and F10 text.
