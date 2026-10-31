```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each proposal the fixer carried out as proposed (F1–F6) by building a case where the proposal is wrong or breaks something elsewhere.

no findings

I couldn't break any of the six:
- **F1:** with the default `plan_hard_max_bounces: 0`, `caps.HardKey` resolves to `review_hard_max_bounces`, and `perchHardCap` now routes it through `ConfigSpec()`. A local `review_hard_max_bounces` at or above loom's `review_max_bounces: 3` passes `LoadTapestryConfig`'s `CheckResolved`. The existing "a local run-wide cap" row already writes the local layer through `configengine.LocalFile` and asserts `--layer local`, so the new row's raise text holds.
- **F2:** `bootstrapMessage` (`internal/shedverbs/step.go:82`) appends `armWayForward` only when the text has no `way forward:`. The appended re-seed clause therefore replaces the generic one on step and on the loop path, which both go through `bootstrapMessage`, and status prints the error text unchanged. `writeSeed`'s "unknown recipe" assertion (`seed_test.go:98`) is untouched, because the clause is added in `armFromSeed`, not in `lookup`.
- **F3:**
  - Moving `TestConfigTemplate_ContainsEveryConfigYAMLTag` to card 6's Edit list is needed. Its loop has no skip for `-`, so `containsConfigKey(text, "-")` would fail (`config_test.go:1129-1134`).
  - The `verify_attempts:` anchor survives card 5, because darn's template keeps that line.
  - Routing each key in `seedLoomConfigWithKeys` by the template that holds it keeps all current callers (`circling_test.go:202,226`, `landingdeps_test.go:528`, `wiring_test.go:593,825,908`) compiling unchanged.
- **F4:** the tail subtest of `TestRouting_ProgressAt` asserts `OnDone`, `OnStuck` and `OnReroute` per row (`progress_test.go:61-68`), including Finalize's reroute to Parent-Align, so removing the new test loses no edge.
- **F5:** the arrow matches the Intent's error result.
- **F6:** `wiring_beam_test.go:114-160` asserts `EscalateJudgedFirst` for tapestry, darn and pawl, plus pawl's nil `TraceFailure`. The new test keeps only the tapestry `TraceFailure` fact.

Tool calls:
1. Read `TestPerchHardCap` rows (`circling_test.go:187-250`): my context showed only its name, and I needed its layer seeding and raise text to test F1's new row.
2. Grep for `armWayForward` and `ReportLoopArmError`: my context lacked how step and loop append the generic way forward, which decides whether F2's appended clause replaces it.
3. Read `bootstrapMessage` and `armWayForward` (`step.go:60-88`): to confirm the generic clause is skipped when the text already names a way forward (F2).
4. Read the budget defaults in loom's `template.yaml`: to check that a local run-wide hard cap in F1's new `TestPerchHardCap` row passes `CheckResolved` against the default soft cap.
