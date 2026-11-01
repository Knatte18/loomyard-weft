# Fixer-report, round 1 (code)

## Changes

- F1: `contracts/stencils/orch/orch-template-usage-limit-wake.md` now tells the orch to wait for the notices of the runs that stopped, to read a filtered `lym batten status <slug>` only for a stop a notice names, and to handle that stop as its role says.
  The line still renders to one line.
  The final clause differs from the proposal, which kept "restart what stopped": the role gates a restart behind investigating the stop (`orch-role.md:47`), so the clause now defers to the role.
  Self-check follow-up: `internal/orchengine/doc.go` described the wake line as telling the orch to check each run, and now matches the stencil.
- F3: `contracts/orchrole/orch-role.md:269` says the attach command is for the operator to type and never for the orch to run.
- F4: `contracts/stencils/registry_test.go` gains an assertion in `TestRegistry_DefaultsAndRelPathAreConsistent` that `Registry().Names()` lacks `orch-template-role` and `Registry().Default` reports no entry for it.
  No new test function.
- F2 (`decision` class): not fixed, not disputed, not deferred; it went to the parent, who recorded the decision (creel #259).

`lym gate test ./contracts/stencils ./internal/orchengine` passes.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, stale wake-line description at `internal/orchengine/doc.go:433`; fixed as proposed.
- fix-proposal: LOW, "restart what stopped" contradicts the role's gate on resuming; fixed with an own change (the clause defers to the role) rather than a new restart rule.
- fix-proposal: LOW, no fallback when no notice arrives; disputed, every fix worse (see Disputed).
- fix-affected-set: no findings.

## Disputed

- fix-proposal LOW 2, ground `every fix worse`: the premise holds, but a fallback for a missing notice means the orch checks runs on its own, which the role forbids (`orch-role.md:55`: "never read `lym batten status` on your own to check on runs").

## Deferred

Nothing deferred.
