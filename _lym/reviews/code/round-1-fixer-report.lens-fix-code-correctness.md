```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

Task restated: correctness check of the fix's code changes through the fix-code-correctness lens. This covers `runAgent`/`prepare` in `verify.go`, the fixture `VerifyTable` fake, the two-row resume test, and `wiring_test.go`.

- **`runAgent`, fresh spawn:**
  - `tables()` runs before `Probe` and `prepare`, so `interactive` is set from the spawning table before `ResetRuns`. `interactive` is true when the config sets `verify_interactive`, and the recorded mode now equals the spawned table's `Interactive`.
  - If `tableFor` fails, `Call` exits at "build table" before `prepare`, so the zero-table `false` is never recorded.
  - `prepare` no longer calls `d.Config()`, so a config error there is gone. The `Head()` error path is unchanged.
- **`runAgent`, resume:** `prepare` is skipped, so `interactive` is set but unused and the counter is untouched. The `MultiLLMProducer.Call` code in context confirms this (`multillm.go:106-111`).
- **Callers of `ResetRuns`:** the signature is unchanged. The other callers are the tests and the capture integration test, which pass a literal.
- **Fixture `VerifyTable` fake:** it now reads `f.deps.Config()` at call time, so tests that replace `Config`, such as the operator-acceptance test, still drive `Interactive`. Without that change the F3 fix would have broken the operator-acceptance test, because the old fake never set `Interactive`. No fixture test makes `Config` return an error.
- **Resume test with the defect in place:**
  - If a resume rewrote the counter, `prepare` would call `ResetRuns(…, interactive=false)`.
  - The operator row would then read `operatorAtResume` false and a count of 0, and fail.
  - The autonomous row would fail on the count of 0, so both rows catch the defect.
  - `ResumeFn` is wrapped after `newVerifyFixture`, and the fake `Resume` dispatches through it, so the capture runs. `CloseAgentSession` keeps the count, so `RunCount` is 1 after the run.
- **`wiring_test.go`:** with `OperatorSession` dropped from `verbStatus`, the operator row gets `false` and fails. With a `ResetRuns` mode that is not honoured, the autonomous row still passes, which is fine because that row pins "false when not operator". The tier-1 gate passed both rows.
- **`autonomousVerifyJudging` reworded:** the tier-0 gate passed, so no test pinned the old wording.

Calls made: none. My context already held every changed file.
