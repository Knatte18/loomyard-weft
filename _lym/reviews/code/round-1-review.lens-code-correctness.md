```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each changed function, branch and stencil instruction with one concrete input.

**List**

- `DarnTable` / `makeChangeList` (`internal/darnengine/spec.go:120,155`): the diff changes no executable line here. Ran `GateFixed=true` and `GateFixed=false`: the gated and ungated stencils are picked as before. Correct.
- `TestDarnSpec` constants (`internal/darnengine/spec_test.go:76-77`): `gatedNoGateStep` and `gatedReproduceRun` are exact substrings of the gated fragment's lines 15–16, and neither appears in the ungated fragment. The ungated row asserts both absent and the gated row asserts both present. Correct.
- `TestWire_DarnRecipe` constant (`internal/loomcli/wiring_darn_test.go:266`): matches line 15 of the gated fragment; the present/absent logic is unchanged. Correct.
- Gated fragment item 3, a reproduce run that fails, e.g. `lym gate test ./internal/x -- -run TestX` with TestX red: the detached runner's `runWithFixer` calls `handsOff`, which hands off only when the call carries `--gate` (`internal/gatecli/cardfixer.go:81-84`). This call has none, so no cold fixer starts or touches the tree, and the writer sees the real failure. Correct.
- Gated fragment item 3, a reproduce run that answers `done: false` (exit 75): the gate rule the item points to says to run the same command again, which joins the same run. That does not break "once". Correct.
- Gated fragment item 3, a reproduce run of a test above tier0, e.g. a tier1 test in `internal/shedverbs`: **incorrect**, see finding 1.
- Gated fragment item 3, a test name that is a prefix of another test, or a subtest name: unanchored `-run`, see finding 2.

**Findings**

1. **BLOCKING · design** · `contracts/stencils/darn/darn-fragment-make-change-gated.md:16-19`
   - **Summary:** the reproduce-run instruction names only the package and `-- -run <TestName>`, never the test's tier. For a test above tier0 the run skips it, passes, and the writer then follows "when it passes" to a false "landed work overtook the entry" stop.
   - **The run, step by step:**
     - The writer runs `lym gate test ./internal/shedverbs -- -run TestLoopX` as written.
     - Without `--tier`, `--upto` or `--gate` the call runs tier0 only (`internal/gatecli/cli.go:75`, `:183-184`).
     - `internal/shedverbs` holds tier0 tests, so `tierTestTargets` keeps the package (`internal/gatecli/tierrun.go:382-392`).
     - `go test -run TestLoopX` under `LYM_TEST_TIER=0` reaches `tiergate.Require(t, 1)`, which skips the test (`pattern/PATTERN-test-speed.md:23`). `go test` exits 0 and the gate passes.
   - **Why the case is live:** a Go tier carries no ran-no-test notice. `noTestNotice` returns empty for any counter other than `counterVSTest` (`internal/gatecli/tierrun.go:367-370`).
   - **Why it is the motivating case:** the incident's own package holds tier1 tests: `tiergate.Require(t, 1)` appears 5 times in `internal/shedverbs/*_test.go`, in `loop_arming_integration_test.go` and `loop_unix_integration_test.go`.
   - **Why the gate rule does not cover it:** the system prompt's gate rule ("use its --tier flag for a tier") is generic, and the item lists the call's parts without the tier.
   - **Fix:** the item names the tier that the test's own `tiergate.Require` gives, with `--tier <n>`. Alternatively, it tells the writer that a pass counts only when the run's output shows the test ran rather than skipped.
2. **NIT · design** · `contracts/stencils/darn/darn-fragment-make-change-gated.md:17`
   - **Summary:** `-run <TestName>` is an unanchored regexp, so `-run TestFoo` also runs `TestFooBar`. A failure of a different test can then read as the reported failure reproducing.
   - **Fix:** anchor it as `-run '^<TestName>$'`.

Out of my lens, noted only: the `GateFixed` field comment (`internal/darnengine/spec.go:67-68`) attributes the one-run exception to "the writer and its subagents", but the stencil grants it to the writer alone.

**Tool calls**

1. `grep` for `test_targets` in gatecli and gateslot: my context lacked how explicit targets narrow to the tier's packages.
2. `Read` of `internal/gatecli/tierrun.go:265-294`: my context lacked `runInvocations`' narrowing and notice behavior.
3. `grep` for `tiergate.Require` tiers in `internal/shedverbs` and `internal/sheddriver`: to check whether the incident's package holds tests above tier0.
4. `grep` for `noTestNotice`: to check whether a Go tier reports a run whose `-run` test was skipped.
