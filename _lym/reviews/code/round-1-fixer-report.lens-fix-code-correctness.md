```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the fixer's two test changes (F1 and F2).

**F1, `check_test.go` row `FailedAfterAGreenTierWithNoFailedTierNamesBeamStatusAlone`.**
- Input: a gate that ends tier0 as passed and exits 1. This is the "gate exits 1 after a passed tier" case.
- The record ends `checkFailed` with one tier and `FailedTier == nil`, which is the new `checkMarkOf` branch at `check.go:333`.
- Expected marks: starting, tier0 in flight, tier0 green 40s, then the failed mark. That matches the row, and the tier0 gate run passed with it.
- Without the branch, that record skips the `Running` and `len(parts) == 0` cases and reaches `default`. The row would then see a `checkRunning` mark, so the row fails with the defect in place.

**F2, `TestRunCLI_Down_IsNotRefusedInBolt`.**
- The guard at `cli.go:166` fires only for `attach` and `up`, so `down` in Bolt prints no refusal and the test passes.
- If the guard were widened to `down`, the output would contain `fabricRefusalText` and the test would fail. The test is not vacuous.
- The assertion is on raw `out.String()`, so empty output or a non-envelope output cannot break it.
- `fabricRefusalText` matches the prefix in `cli.go:168` ("boots a session only in a worktree of the hub"). The existing refusal test already asserts the longer prefix, so the const and the guard text cannot drift apart unseen.
- No caller or signature changed; both edits are test-only. The tier0 gate was green, and the earlier tmux-leak failure came only from the removed `attach` and `up` rows.

Findings:
- LOW: part of F2's `fix` was not made. The `attach`/`up`-pass-the-guard row for a task worktree is absent. Running them starts a real tmux server, which tier0 forbids. It is not a correctness defect in the changed code, and the parent should record it as a deferral with that reason. The smoke tests are tmux-tagged and call `mustRunReed(prime, "up")`, so `up` acceptance in a prime is covered there; `attach` acceptance is not covered by any test.

No tool calls beyond the context.
