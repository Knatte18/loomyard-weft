```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals as carried out (fix-proposal lens).

Findings:

1. **LOW** — F6, `internal/battencli/testmain_integration_test.go` `TestMain`.
   - The refusing seam is installed whenever the selection is exactly tier0. A single `lym gate test` run does not select a lone tier0 for the whole module. A tier1-only battencli test run under `LYM_TEST_TIER=1` gets the real push, which is intended.
   - The proposal does leave a hole. Under `--upto 2` the gate runs tier0, tier1 and tier2 as separate invocations, so tier0 stays refused. A developer's `LYM_TEST_TIER=0-1` run gets the real push for tier0 tests too.
   - A tier0 test that reaches `wire.go:980` or `:1425` would push for real in that mixed run, and the `TestMain` comment says nothing about this. The refusal protects only pure-tier0 runs, so it is weaker than the removed `!tier1` stub.
   - The proposal is acceptable, but the comment "so a tier0 test that reaches a pair push fails" overstates it. Qualify it with "in a run selecting tier0 alone".

2. **LOW** — F5, `cmd/lym/tierpurity_test.go` `TestTierPurityFailures`, row `Tier2FileReferencingTmuxkitSpawn`.
   - The row's source is `tiergate.Require(t, 2)` plus `tmuxkit.Socket(t)`, and it expects no failure. This passes because `FileTier` returns 2 and `tier < tmuxTier` is false. It would also pass if the whole tier>0 branch were deleted, so the row does not pin the `tier < tmuxTier` comparison.
   - The tier1 row does pin the comparison, so the coverage gap is partial. No change is needed.

3. **LOW** — F5, `cmd/testtiming/spawnscan_test.go` fixture `fixtureRequireTier2Tests`.
   - The fixture imports `github.com/Knatte18/loomyard/internal/testkit/tiergate`, which is not under `fixtureModule` (`example.com/m`). The expectation `outOfProcess` relies on `FileTier` reading the Require, and I have no evidence it would fail otherwise.
   - If `FileTier` ignored the Require, the file would read tier0 and `TestRequireTier2` would be judged (`{}`). So the row does fail without the Require reader, which is the point. No defect.

4. **LOW** — F3, `internal/testkit/tiergate/tiergate_test.go` `recordingTB`.
   - It embeds a nil `testing.TB`, so any further `t.*` call inside `Require`, such as a future `t.Logf`, panics with a nil dereference instead of failing cleanly. This is acceptable for a fake that is bounded to `Require`'s current calls.

5. **LOW** — F1, `crucible/orchestrator-prompt.md`.
   - `export LYM_TEST_TIER=2` sits mid-block, so it persists into the "teardown check" line and any later command the operator pastes from the same block. The reviewer's proposal asked for the export, but a subshell or an inline `LYM_TEST_TIER=2 "$SCRATCH/smoke.test.exe"` would scope it. The leak is harmless here.

Calls made: none (the context already held every changed file).
