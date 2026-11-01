```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the fixes for F1 and F2 cover every member of the sets their premises name (the lens is fix-affected-set).

Unchecked findings: none.

**F1 (callers of `liveProcesses` and `inspectHub`):** no findings.
- The grep found one production caller of each, both in `main.go`. Both use the new signature or the resolved path: `liveProcesses(hubPath, absPath, ...)` at line 53 and `inspectHub(hubPath)` at line 57, where `hubPath` is now the `EvalSymlinks` result.
- The test calls match the new signature.
- `main` later builds its printed paths from `hubPath`, which is now the resolved path. That is consistent.

**F2 (stop branches in `inspectHub`):** no findings.
- The review named four untested stops: a stray file, no prime with its weft, a wrong checkout `.git` link, and a pre-existing new admin dir. The fix added a row for each.
- It also added the own-PID skip row to `TestLiveProcesses`.
- One stop branch is still untested: the check on `<checkout>/.lym/runproc/<new name>` in the `checks` map (`main.go:149`). The review did not name it, so I did not count it as a finding.

Further tool calls:
1. `grep -rn "liveProcesses(\|inspectHub(" --include=*.go .`: to list every caller of the two changed functions, which my context lacked.
