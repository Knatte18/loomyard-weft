```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the F1/F2 test and stand-in changes for correctness, through the fix-code-correctness lens.

No findings.

Inputs I ran step by step:
- **Stand-in orphan flag (`runStandIn`):**
  - With `helperOrphanEnv=build` and a `sharedbuild` invocation, `orphan` is true and the sleeper starts holding stdout. The record carries `Grandchild`, and the stand-in prints the failure line and returns 1.
  - With `helperOrphanEnv=build` and a tier invocation, `orphan` is false, so the tier row is not disturbed.
  - With `helperOrphanEnv=tier` and a prebuild (`build`) invocation, `orphan` is false, as before.
  - If the sleeper fails to start, the stand-in returns 1 before it writes a record, so the build subtest fails at its `len(builds) != 1` check instead of passing on a zero pid.
- **F1, `ZeroExit` row:** the stand-in exits 0 and the orphan holds the pipe. `Wait` returns `exec.ErrWaitDelay`, and `runTiedChild` maps it to nil, giving exit 0 and outcome `passed`. With the nil mapping removed, the caller gets a non-`ExitError`, so the exit code is not 0 and the row fails. A run showed the row passing.
- **F2, build subtest:** `runBuild` passes `e.request.pipeWaitDelay`, which `helperPipeDelayEnv` sets to 100ms. The sleeper is killed by `killLeftovers` through the process group. The build record is `failed` with exit 1 and `TimedOut` false, and the call exits 1. If `runBuild` passed a zero delay, `Wait` would block on the orphan's pipe and the call would hang, so the test would fail through `g.wait`'s bound. A run showed it passing.
- **Other checks:**
  - `helperOrphanEnv` has no other user, so the `1` to `tier` rename leaves no caller broken.
  - Loop variable `tt` is captured safely, since Go 1.22 or later scopes it per iteration and the subtests are not parallel.
  - `strconv` is already imported.
  - `lym loom fmt` reported no changes.

Tool calls: none.
