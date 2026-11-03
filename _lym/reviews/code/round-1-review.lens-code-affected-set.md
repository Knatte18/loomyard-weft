```yaml
lens: code-affected-set
outcome: reported
```

**Lens code-affected-set: affected set per change**

1. **`runTiedChild` signature (new `pipeWaitDelay` parameter): covered.**
   - The set is its only two callers, `internal/gatecli/tierrun.go:228` and `internal/gatecli/build.go:47`. Both pass `request.pipeWaitDelay`. A grep of `internal/` found no other caller.
   - The detached card-verify line runs through `verifyrun`, not `runTiedChild`. `detachfront_integration_test.go:964-989` expects a backgrounded `sleep` to outlive a killed runner. The new `killLeftovers` does not touch that path, so the test's expectation holds.

2. **`newCommand` signature (new `pipeWaitDelay` parameter): covered.**
   - Callers: `cli.go:35` (`Command`), the `runnerCmd` rebuild closure at `cli.go:121`, `detachrun_test.go:67` and `test_integration_test.go:156` (`runGateHelper`). All pass the new argument.
   - `newCommand` is unexported, so no other package can call it.

3. **`testRequest.pipeWaitDelay` (new field): covered.**
   - It is set in `cli.go:90` and `rungate.go:28`.
   - The only other constructors are the zero-value literals in `tiers_test.go`. They run no child, so a zero `WaitDelay` is harmless there.

4. **`defaultPipeWaitDelay` (new constant): covered.**
   - It is declared once in `child.go:18`. Production reads it only at `cli.go:35` and `rungate.go:28`.
   - The test helper default at `test_integration_test.go` references the constant rather than repeating the literal (`PATTERN-named-constants`).

5. **Behavior change: the leftover kill now runs after every exit, and `ErrWaitDelay` after a zero exit maps to nil.**
   - The set is `exitCodeOf` (`run.go:632`), the exit and outcome recording in `tierrun.go` and `build.go`, and the tests that spawn a sleeper.
   - `exitCodeOf` gets a nil error, so a zero exit records 0. A failing exit keeps its `*exec.ExitError`, so its code is recorded.
   - The other sleeper tests hold their stand-in with `select {}`. The tier ends by cancel or signal, so the leftover kill does not change what they assert.
   - Covered.

6. **Docs: covered.**
   - The package doc's Child lifetime section and the `newCommand` parameter paragraph in `doc.go` both land in this diff.
   - `docs/dotnet-hub.md:150` says a call "waits on idle nodes until `timeout_min`". That describes a shell still running, so the change does not make it wrong.
   - `docs/overview.md:501` (the gate paragraph) states no child-lifetime behavior, so nothing there goes stale.
   - `docs/benchmarks/test-redundancy.md` is generated, and the diff adds a subtest, not a test function.

7. **PATTERN entries and `pattern/` background files: covered, no finding.**
   - The diff declares no PATTERN changes, so there is nothing to land.
   - `PATTERN-spawn-observability`: `killLeftovers` and the `ErrWaitDelay` branch log through `internal/logger`.
   - `PATTERN-test-speed`: the delay is injected, not a package variable, and the new subtest passes 100ms.
   - `PATTERN-gate-slot`: the change keeps the slots held, and the kill runs before they are released.
   - `PATTERN-run-process-owner`: `killLeftovers` ends the gate's own tier child, which no `runproc` record names. `detachfront.go:655` already calls `proc.KillTree` on the same kind of child, so I raise nothing.

8. **Commit message: covered.** It carries no `Pattern-Departure:` trailer, and none is needed, since no entry forbids the way taken.

**Findings:** none from this lens.

**Tool calls**
1. `grep` over `_bolt/_lym/specs` and the repo `docs/`, `README.md` and `CLAUDE.md` for gate child-lifetime wording. My context held only `doc.go`.
2. `grep` over `internal/gatecli/*_test.go` for tests that expect a leftover to outlive its shell. My context held only the new orphan test.
3. `Read` of `detachfront_integration_test.go:930-990`, to check that the survival test at 964 goes through `verifyrun` and not `runTiedChild`.
4. `grep` over `docs/dotnet-hub.md` and `docs/overview.md` for MSBuild leftover and child-lifetime claims. My context held neither file.
