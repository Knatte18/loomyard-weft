---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/shuttlecli/smoke_bashstdin_test.go:76
    summary: the smoke test passes vacuously if the agent never attempts git reset --hard, so it does not pin that the deny rule matched the prefixed command
---

Range: `git diff $(git merge-base main HEAD)..HEAD`, no rework round exists.
`CGO_ENABLED=1 go build ./...` and `go test` over the touched packages pass, and `go vet -tags smoke ./internal/shuttlecli/` is clean.

Focus questions settled without a finding:

- Card 4: `bashStdinRewriteJSON` carries `updatedInput` and no `permissionDecision`, and the hook is appended after the fork guard.
- Cards 8 and 11: `Fabric.ResetPairWarp` runs the pipeline's ownership, dirtiness (`dirtyTrackedExcept`) and force checks through `resetHardTo`, and `PlanReset` refuses every unsafe target before the CLI calls fabric.
- Card 9: `recordPreFix` runs at the closure's first failure and `clearPreFix` on each passing evaluation of a done outcome; the two early passes (outcome not done, outcome unreadable) verify nothing and leave the value, which the plan's "on a passing evaluation" does not cover, so it is not a defect.
- Cards 12 and 15: every reset-naming refusal names `lyx webster reset --to start|pre-fix` or `git reset --keep <sha>`, and the scan's self-test plants a denied form and asserts a hit.
- Cards 13 and 7: `contractFileStatus` clears only on absence or a Master write later than every fork write, by the write events' time and result.
- Card 14: `pendingPathsWayForward` returns ordered steps rendered once through `wayForwardSteps`, ending in one re-entry step on both routes.

### [LOW] Smoke test cannot fail when the denied command is never tried

**Location:** internal/shuttlecli/smoke_bashstdin_test.go:76
**Issue:** The test asserts the dirty file is unchanged and the run is done.
An agent that skips `git reset --hard HEAD` satisfies both, so the card's pin that the deny rule still matches the prefixed command can pass without the rule ever being exercised.
**Fix:** Have the prompt ask the agent to append `REFUSED` or `RAN` to the output file after the reset attempt, and assert the file ends in `REFUSED`.

No `## Focus departures`: none taken.
