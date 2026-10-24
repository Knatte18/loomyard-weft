```yaml
lens: code-tests
outcome: reported
```

Task: lens code-tests. For each behavior the diff changes, which test pins it.

**Per behaviour**

- Failed tier doesn't stop the next tier; exit code is the first failure's, a later different failure keeps it; closing line names every failed run: pinned by "a failing tier stops no later tier and the call exits with the first failure's code" (`test_integration_test.go:939`). It asserts exit 7, the tier6 record with exit 9 and `failed`, and the exact closing line.
- No closing line when every run passes: pinned by an assertion added to the passing two-tier subtest (`test_integration_test.go:929`). The assertion is a substring check on "failed runs". That's fine.
- Guard step runs after a failed listed tier and exits with the listed tier's code: pinned by the guard-table row "a failing tier still runs the guards and returns its code" (`test_integration_test.go:1344`). It asserts exit 3 and invocations `{./pkg}, guardRun`.
- Closing line names a guard run (`guard tierN`): none. No test fails a guard run, so the `guard ` prefix in `runLabel` (`run.go`) is unpinned. The row at `:1344` fails only the listed tier, and the guard run passes. The first-failure rule across a listed failure and a guard failure is unpinned for the same reason.
- Timed-out tier counts as failed and doesn't halt: pinned by the `--card 1` scope test (`scopes_integration_test.go:~220`). It now asserts the guard invocation runs after the timeout and that the exit is 1. Its assertion on the tier0 timeout line still holds.
- Refusal halts the call with a single JSON envelope: pinned by the existing refusal table (`test_integration_test.go:1215-1250`, about 1229 and 1245) and the JSON-envelope check near `:642`. I did not see a refusal in a later tier after an earlier tier had run (the mid-run case). Mid-run refusals are only partly pinned, and the diff adds nothing for them.
- Busy hub ends the call with exit 75: pinned by "every slot held past the wait cap answers slot_free false…" (`:630`). It is a first-run case, not a later-tier case. A busy hub on the second tier, after an earlier failure, is unpinned.
- Signal halts with 128+N: pinned by "a catchable signal kills the tier's whole group…" (`:988`). It uses a single tier, so `report`'s `halt: e.ctx.Err() != nil` path is exercised only on the first run. A signal arriving during a later tier or a guard run is not pinned.
- Carried or empty tier reports exit 0 without halting, and a pass is recorded only for `end == runEnd{}`: the pre-existing carry tests (`:507-514`, `:563`) cover the carried path and are unchanged. No test pins that a failed tier doesn't carry on the next call.
- `runGuard` error path (`guardStepRun` failing) halts: the refusal table covers guard-related refusals before any slot, but I could not confirm one that reaches `runGuard`'s error branch. Treat it as unpinned unless the handler finds one.
- Removal of `guardRunOf`: no test needed.

**Findings**

1. LOW, class `design` (coverage gap), `run.go` `runLabel` and the closing line, evidence `test_integration_test.go:1344` and `:952`. A failed guard run, its `guard tierN` label in the closing line, and the first-failure rule across a listed failure and a guard failure have no test. Adding an assertion to the `:1344` row or the `:939` subtest would pin them. The row only needs a guard run that also fails, and the acceptance criterion "names every failed run, listed tier and guard run alike" is only half covered.
2. LOW, class `design` (coverage gap), `run.go` `report` (`halt: e.ctx.Err() != nil`) and `acquireSlot`, evidence `:630` and `:988`. The halt cases (busy hub, signal) are pinned only on the first run. The new behavior is stopping after a failed earlier run, and that interaction is unpinned. The acceptance criterion says they "still end the call at once, as the existing tests pin", so this is minor.
3. No test-economy violations. The changed tests extend existing subtests and table rows, and no new test duplicates another. Stale names and messages in the changed tests are updated. I found no per-helper test, review-fix test or stencil-prose assertion.

**Tool calls (2)**
1. `grep` of subtest names in `test_integration_test.go`: my context lacked the full list of subtests, to see which halt cases are pinned.
2. `sed -n` over `test_integration_test.go` lines 972-1030, 1215-1230 and 1352-1366, plus a grep for "failed runs": my context lacked the timeout, signal, refusal-table and guard-table bodies, and I wanted to see where the closing line is asserted.
