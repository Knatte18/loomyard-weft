---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: internal/shedverbs/loopwait_test.go:49
    summary: the diff drops the wait-world fixture's 20 ms lock grace, a flake fix the amended board entry does not ask for
    fix: Leave the code in place and defer to the run's parent. The change is test-only, the change description names its reason, and reverting it would bring back the intermittent loop-wait failures. The parent can record it with lyx loom decision add or split it into its own task.
    origin: handler
  - id: F2
    severity: NIT
    class: consistency
    location: internal/shuttleengine/claudeengine/settings_test.go:742
    summary: the new Contains row asserts shell-guard prose that the regenerated settings-golden.json comparison already pins
    fix: Remove the added row "When it answers that no gate slot is free yet, run the same command again in a turn that does nothing else, so the prompt cache stays warm between calls" from the want list in TestBuildDenyNotice_SentenceContent. The golden-file comparison in the same test file covers the wording.
    origin: lens:code-tests
---

### [LOW] Unlicensed shedverbs fixture change

**Location:** internal/shedverbs/loopwait_test.go:49
**Issue:** The amended board entry covers the gate slot wait, the retired key, the stencils, the docs and testtiming. It does not cover the `internal/shedverbs` loop-wait fixture. The change description gives a reason (intermittent failures under full-suite load) and `LockGrace` zero selects `loopLockGrace` in `loop.go:170`, so the edit is correct. It is still scope the entry does not license.

### [NIT] Redundant stencil-prose assertion

**Location:** internal/shuttleengine/claudeengine/settings_test.go:742
**Issue:** `PATTERN-test-economy` flags a new assertion on static prose. The regenerated `testdata/settings-golden.json` already fails if the gate rule wording changes.

## Rejected

- Nothing pins that `Command` passes `DefaultAwaitCap` (lens:code-tests, F-T1): the help subtest in `cli_test.go` runs `RunCLI`, which builds `Command()`, and the help text formats the `slotWait` it was given, so a copied or wrong value fails it. The `cli_test.go` header was widened to say so.
- No test asserts that `ConfigSpec().RetiredKeys` holds `cli_wait_sec` (lens:code-tests, F-T3): the "a retired cli_wait_sec still loads" row passes through `configengine.Load`, which only tolerates the key through `RetiredKeys` (`layers.go:155-180`), so the row pins the registration.
- No test covers the `testtiming` slot wait (lens:code-tests, F-T4): the wait is tooling, a stuck holder ends it through the same context timeout and error path, and a 250 s test is not reasonable.
- The `settings-golden.json` diff (256 lines): only the gate rule sentence changed, with no unrelated churn.
- Retiring `cli_wait_sec`: a repository-wide grep finds the key only in the retirement code, its doc and the load test, and a hub file that still sets it loads.
- The stencils: the diff changes only the busy-answer wording and drops no other rule.
