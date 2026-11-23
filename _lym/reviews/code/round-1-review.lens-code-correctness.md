```yaml
lens: code-correctness
outcome: reported
```

Task: review the diff's changed functions and branches through the code-correctness lens.

**Per function or branch: the case I ran and the result**

- **`redFirstGate.verify`, new exit-5 branch** (`internal/loomshed/redfirstgate.go:216-219`):
  - **Plain `exit 5`:** `verifytree` returns `StatusFailed`, `ExitCode` 5, with `TimedOut` and `RunnerCrash` both false (`internal/verifytree/verify.go:437-440`). That skips the status switch and takes the new branch, which logs and returns `Passed: true`. Correct.
  - **`exit 5` with a tier-timeout line in the log:** `TimedOut` is set, so the branch is skipped and `verify` returns a nil verdict. Correct.
  - **`exit 5` with the runner-crash line:** `RunnerCrash` is set and the result is the same. Correct.
  - **Shell could not start:** `ExitCode` is -1, so the branch does not apply. Correct.
- **`arrival` with exit 5:**
  - **No hold:** the verdict is not nil, so `heldTree` is cleared and the gate passes. Correct.
  - **Hold on tree T from an exit 4, then a new commit T2 that exits 5:** the tree differs, so `verify` runs, the gate passes and `heldTree` is cleared. If a later run exits 4 again, the `heldTree == ""` path resets `told` and the line is offered afresh. Correct.
  - **Exit 5 with a timeout or crash:** `holdsForCommitOrRecord` is false, so `redFirstFailure` handles it through the `TimedOut`/`RunnerCrash` cases and the result is terminal. Correct. The `exit 5` timeout row covers this.
- **`final` with exit 5:**
  - **No hold:** `verify` passes. Correct.
  - **`heldTree` set by an exit-4 hold on the same tree:** it returns the exit-4 failure from the hold without running anything. That is correct: an unchanged tree can only switch to exit 5 if the hub's scoper config changes in the middle of the run, which no realistic path does.
- **`holdsForCommitOrRecord`:**
  - Exit 4 gives true and exit 5 gives false, both correct.
  - Exit 4 with a timeout gives false, which leads to a terminal failure. Correct.
- **`redFirstHoldLine`:** it is reached only when `holdsForCommitOrRecord` is true, so only for exit 4. Dropping the exit-5 wording loses no reachable case. Correct.
- **`redFirstFailure`, exit-5 case removed:**
  - A failed exit-5 result now reaches this function only with `TimedOut` or `RunnerCrash` set, and those switch cases come before the default.
  - The `heldResult` passed in from `final` can no longer be exit 5, because the hold is held in memory and only exit 4 sets it.
  - So the default "other exit code" branch is unreachable for exit 5. Correct.
- **`runRedFirst`, non-go scoper branch** (`internal/gatecli/redfirst.go:128-131`): only the message changed. It still returns `RedFirstUnjudgedExit` with an `ok:false` envelope. Correct.
- **Can exit 5 come from anything other than the non-go scoper?** No. `redFirstCall.run` returns:
  - 0
  - `RedFirstGreenExit` (3)
  - 1, through `output.Err`
  - 128 plus the signal number, through `exitAfterSignal`

  The early refusals return 1 through `output.Err`. Within the verb, exit 5 is reached only from the non-go scoper branch, so the gate does not pass any other failure by mistake. Correct.

**Findings:** none.

**Tool calls**
1. Grepped `internal/verifytree/*.go` for how `TimedOut`, `RunnerCrash` and `ExitCode` are set on a failed result. My context had the gate's use of these fields but not how they are set.
