---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    origin: handler
    location: internal/gateslot/pool.go:356
    summary: "The diff adds a `draining` field, `Pool.Draining` and a stderr note for skipped manual tiers, none of which the amended board entry asks for"
    fix: "Keep the code. The decision record, the writer's own claim, licenses these, and they serve the entry's goals of showing why a run waits and why a tier did not run. Leave them in place and let the parent decide whether to record them with `lyx loom decision add`. Do not revert."
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/test_integration_test.go:515
    summary: "No test pins `lyx gate status` reporting `draining: true`; the scenario asserts only the false case, so a hard-coded false in `runStatus` would pass"
    fix: "In the `status names each held slot's number…` subtest of `TestGateTest_Scenario`, or in a new row beside it, hold the drain lock with `lock.TryAcquireWriteLock` on the pool's drain lock path and assert `envelope.Draining` is true, then release it."
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/gatecli/cli.go:55
    summary: "The `test` help text says the default is tier0 and does not say that `--upto` alone replaces that default with tiers 0 through N"
    fix: "In the `Long` text of `testCmd`, change the `--tier N` line to say the default is tier0 when neither `--tier` nor `--upto` is given."
---

### [LOW] Added surface beyond the board entry

**Location:** internal/gateslot/pool.go:356, internal/gatecli/tiers.go:182
**Issue:** The amended entry asks for `--upto`, `manual`, and a read-only `status` showing taken slots by run, tier and start time. The diff also adds `Pool.Draining` with a `draining` output field, a stderr note for each skipped manual tier, and a negative-`--upto` refusal. The `CLAUDE.md` push-gate edit is also not named in the entry, but the entry's Seen section names that call as the thing to replace. All of these are small, read-only or guarding, and consistent with the entry's goals. They are filed only because the rubric makes unlicensed additions a finding, and the fix is to defer rather than revert.

### [LOW] `draining: true` untested through the CLI

**Location:** internal/gatecli/test_integration_test.go:515
**Issue:** `TestDraining_ReportsTheDrainLockWithoutCreatingIt` pins `Pool.Draining`, but the status wiring is asserted only with the drain false.

### [LOW] Help text omits the `--upto` default replacement

**Location:** internal/gatecli/cli.go:55
**Issue:** `listedTiers` returns tier0 only when neither flag is given, so `--upto 2` alone runs 0 through 2. The help text's "the default is tier0" line does not say so.

## Rejected

- Pattern-fit nit on the `cli.go:75` example `--tier 0 --tier 1 --all` sitting beside the new `--upto 2` example: the examples are consistent, and no rule is broken.
- Correctness note on `strconv.Atoi` ignoring its error in `Holders`: only the pool writes `slot-*.yaml` names, so a non-numeric name is unreachable.
- Test gap that `lyx gate status` is not pinned as read-only: the read-only property is structural, since the code takes no slot and writes no record, and `Draining` already has a no-create assertion.
- Claim that `--upto 2` runs the same tiers as the old push gate: it holds for a hub that declares tiers 0, 1 and 2, which is the loomyard hub, so the summary is borne out.
