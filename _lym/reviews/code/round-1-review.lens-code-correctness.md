```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff. I found one defect. Everything else I ran checks out.

**Findings**

- **F-A, LOW, consistency.** `internal/shuttleengine/wait.go`, in the `finalizeAndRemoveDiedWindow` hunk (new lines about 2052–2067). The diff inserts `gateEndedEarly` between the doc comment "// finalizeAndRemoveDiedWindow finalizes the run with outcome, then ends the window…" and its function. That comment now leads `gateEndedEarly`'s doc comment, and `func (run *Run) finalizeAndRemoveDiedWindow` has none. Fix: move `gateEndedEarly` and its comment above the `finalizeAndRemoveDiedWindow` comment, or move the comment back onto its function. Evidence: the diff hunk at old line 2048.

**Per function: case run, result**

- `runTest` stop branch (`gatecli/run.go`): the first of three tiers fails, so the call stops and the stderr line names the two skipped. Correct.
  - The last-plan-entry failure (`i+1 == len(plan)`) prints nothing and ends normally. Correct.
  - A blocked build is labelled "build". Correct.
  - A runner crash on both the run and its rerun (`crash != ""`) does not stop the call. Correct.
  - A carried or unrun entry (`ran` false) does not stop the call. Correct.
  - After `break`, lint and report steps still run. Correct.
- `stopsCall`, `stopOnFailLine`: correct.
- `FailedTests` (`gateslot/runlog.go`): I checked the parser against the report builder in `gatecli/failurereport.go`.
  - A tier run header with test lines gives the tests and `ok` true. Correct.
  - A header with no test lines before the next header gives false. Correct.
  - A `lint …` or `vet tierN` header gives false. Correct.
  - A "failed outside any test" line has five fields and gives false. Correct.
  - A cut report's closing line (`... N more lines cut …`) takes the unindented branch and `reportRunTier` rejects it, so the result is false, as the doc says. Correct.
  - An empty log or no header gives false. Correct.
  - Duplicate tests are deduplicated. Correct.
  - Report output lines are never blank (`failurereport.go:97-98` skips them), so the blank-line break cannot end the parse early. Correct.
- `reportRunTier`, `reportTestLine`: a guard header gives its tier; a negative number or a non-`Test` top-level name gives false. Correct.
- `FailedTestsCall`: an empty list errors; a quoting failure errors; tiers are sorted and joined by `&&`; packages and test names are deduplicated. Correct.
- `StopOnFailCall`: appends the flag. The darn wiring applies it only to `darnWriteGateKey` and skips an empty tree-scoped command. Correct.
- `NewVerifyGate` darn path:
  - A named failure fills `failedBefore`; the next arrival clears it before the rerun. Correct.
  - A rerun that fails again returns decided with the findings alone and keeps the tests. Correct.
  - A rerun that passes, or is skipped, falls through to the full command, and `failedBefore` was already cleared. Correct.
  - A dirty tree during the rerun keeps `failed`. Correct.
  - A render error falls back to the full command. Correct.
  - A `Verify` error returns an error. Correct.
  - A timeout, runner crash or unreadable log in the rerun gives `namedFailures` false, so the full command runs. Correct.
  - A full-run failure with no named tests sets `failedBefore` to nil. Correct.
  - A runner crash on the full run is still `Terminal`. Correct.
  - Non-darn forms never touch `failedBefore`. Correct.
- `budgetClock`:
  - `Limit` ≤ 0, `Notify` nil, or a negative minute count from config is off. Correct.
  - `noticed` is set under the lock, so there is one notice. Correct.
  - A failed `Notify` is logged and not retried, as documented. Correct.
  - If the budget is spent between `arrive` and `arm`, `arm` returns a no-op and the notice waits for the next arrival. This is a minor delay only.
  - `timer.Stop` does not wait for a firing callback, so a notice can trail the gate's return by a moment. This is minor and harmless.
  - `verifyBudgetNotice` cuts at a rune boundary, and `cut` is always below `len(line)`. Correct.
  - `failed` falls back to the exit code when the log is unreadable or has no closing line. Correct.
- `gateFailedLead`, `gateFailedReason`:
  - `EndedEarly` empty gives the old wording.
  - `EndedEarly` set with 0 attempts gives "never re-prompted, since …".
  - `EndedEarly` set with N attempts gives the "re-prompts left" wording. Correct.
  - A gate whose entry has `Attempts` 0 hits `gateFails >= Attempts`, so `EndedEarly` is empty and the old "after 0 attempts" wording remains. That is an edge case only a misconfigured `verify_attempts: 0` can reach, so I'm not filing it.
- `gateEndedEarly`: its condition matches `wait.go:590`. `gateEndCause` is set only on paths that finalize at once. A pass, a terminal failure or a spent entry gives "". Correct. A cancelled or died finalize with re-prompts left reads "session ended or deadline", which is acceptable.
- `darnengine.Config`, `ConfigSpec`, `template.yaml`: a `verify_budget_min` of 0 turns the notice off. Correct.

**Tool calls**
1. `grep` in `gatecli/failurereport.go` and `shuttleengine/*.go`, to confirm the report format (cut line, blank lines) and `gateFails` counting, which the context lacked.
