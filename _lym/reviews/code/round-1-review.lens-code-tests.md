```yaml
lens: code-tests
outcome: reported
```

Task: lens code-tests. For each behaviour the diff adds or changes, the test row that pins it, or none.

**Behaviour → pinning test**

1. Explicit targets are refused when no kept tier's command holds `{targets}` or `{test_targets}`: pinned by the `tiers_test.go` row "explicit targets that no listed tier's command takes", which asserts the exact refusal text.
2. The refusal is skipped when at least one kept tier takes the targets: pinned by the row "explicit targets run a tier whose command cannot take them whole, naming it, beside a tier that takes them".
3. A placeholder-free command under a non-target scope still runs, with the notice: pinned by the rows "a command without {targets} runs once, whole, under a narrower scope, naming it" and the dotnet `dotnet test {filter}` row. Both now carry `wantStderr`.
4. The notice moved from `runListed` to `planTierRuns`: pinned by those same `wantStderr` rows. The old integration assertion was deleted, and the planner rows replace it.
5. `unrunProjectLines` and `runReachesProject`:
   - Pinned: a project the whole run does not name, with the lowest takes-targets tier as the way forward.
   - Pinned: a tier whose command names the project, quoted and with backslashes.
   - Pinned: a manual-only tier, which gives the config-step wording.
   - Pinned: a project another run names among its targets.
   - Pinned: a solution file in the command.
   - Pinned: a command with no project file.
   - Pinned: a project reached by `Fast.csproj` while `A.csproj` is reported, which covers the multi-project case.
   - Not pinned: a run with zero invocations, where `runReachesProject` returns false.
   - Not pinned: the same project selected by two runs, which `slices.Compact` merges.
   - Not pinned: case-insensitive file-name matching, and a backslash project path.
6. The `run.go:222-226` wiring has no test. It prints `unrunProjectLines` only under the dotnet scoper, to stderr, with no change to the exit code. Nothing outside the unit test calls it, and the `TestGateTest_Scenario` diff touches only the envelope read.
7. The refusal reaching a caller as a pre-slot error envelope has no end-to-end row. The integration subtest that covered a `--tier 2` run with explicit targets was deleted, and no integration row replaced it. The planner row pins the message, and `run.go:102-105` passes the error straight to `output.Err`.
8. `ignoresTargets` was removed from `tierRun`, along with the ` ignores` marker in `describe`. The `wantStderr` rows now pin that behaviour.
9. `doc.go` is prose, so it is exempt.

**Findings**

- **F1, MEDIUM, class design.** Location: `internal/gatecli/run.go:222`. The dotnet-only emission of the unreached-project lines has no test. This is the "warn when a card adds tests in a project no tier reaches" half of the board entry. The unit test calls `unrunProjectLines` directly. If the `cfg.Scoper == "dotnet"` guard, the `printLine` call or the position after the report were broken or dropped, no test would fail. Fix: add a row in `TestGateTest_Scenario`, or a `runTest`-level test, that runs a dotnet `--card` scope whose tier0 command has no placeholder. It asserts the stderr line, and that the exit code is still 0.
- **F2, LOW, class design.** Location: `internal/gatecli/dotnetscope_test.go:36`. Three branches of the new helpers have no row: the zero-invocation run that reaches nothing, a project selected by two runs that must yield one line, and a project path with backslashes or different case. Fix: add these as rows in `TestUnrunProjectLines`, for example `tier1Run: []invocation{}` with a tier1 that takes targets.
- **F3, LOW, class design.** Location: `internal/gatecli/tiers_test.go:490`. The refusal row has only one kept tier. The `tierNames(kept)` list and the joined multi-tier text are not pinned, nor is the case where `--tier` lists several placeholder-free tiers and all are refused. Fix: add a row with two whole tiers, for example `tiers: []int{0, 1}` with both commands lacking placeholders.
- **F4, LOW, class design.** Location: `internal/gatecli/test_integration_test.go:1801`. The deleted subtest "a command without {targets} runs whole under explicit targets" pinned the process-level result. Its replacement, the pre-slot refusal envelope with exit 1 and no tier command invoked, is covered only at planner level (see item 7). Fix: add one integration subtest `test --tier 2 ./pkg` that expects `ok:false` and exit 1, and no `tierArgs` invocation.

No redundancy or duplicate-coverage findings. `TestUnrunProjectLines` and the new planner rows each pin distinct behaviour. The redundancy report lists the changed tests as "not judged".

**Tool calls:** none. The inherited context already held the diff and every file I needed.
