```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix set (F1–F4) through the correctness lens.

**Findings**

1. **MEDIUM** — `internal/adoptcli/deps_test.go`, `TestAdoptGateTiers`: F1's wiring premise is still unpinned.
   - The test calls `adoptGateTiers` directly.
   - The review's fix said removing the `Tiers` assignment in `hubDepsAs`'s `StartSuite` (`start.Tiers = adoptGateTiers(prime)`, deps.go:158) would fail a row. It would not.
   - `TestAdopt` feeds `Tiers` through its own fake `StartSuite`, so deleting line 158 leaves every test green.
   - Reaching it needs `StartSuite` to run through `ensureCheck` without spawning. A running record with the test's own pid and no start ticks would do that: `checkAlive` reads it alive and returns `AlreadyRunning`, and `Tiers` is then set from a seeded `gate.yaml`.
   - The fixer's report should name the wiring as deferred, or the fixer should add that case.

2. **LOW** — `deps_test.go`: the review's "gate config without the adopt key" row is replaced by "adopt key naming an undeclared tier".
   - The replacement is sound. The gate test log shows `adoptGateTiers` returned `[0]` for a config lacking `adopt`, so the template's default key fills it in and a no-key case cannot exist.
   - The undeclared-tier row does exercise the `TierList` error path (nil result, no panic).
   - The fixer-report should record this as fixed differently.

**Checked, no defect**

- **`adoptFields` (cli.go):** the extraction is behaviour-identical. The error path still passes `fields` to `ErrFields`.
- **`TestAdoptFields`:** it fails if the `message` or `suite_warning` assignment is dropped. It also pins `suite` and the absence of `message`/`suite_warning` in the other rows.
- **Fixture clock (`check_test.go`):**
  - Reads in `runCheck` are in this order: `record.Start` (read 0), `tierStart` in `Started` (reads 1, 2, …), then `EndedAt` in `finish`. The hard-coded `checkFixtureAfter(1)` and `checkFixtureAfter(2)` match.
  - A mutant timing the in-flight mark from `record.Start` yields `checkFixtureNow` and fails the DeepEqual.
  - The failed-before-tier row has no tier marks, so the shifted `finish` read does not matter.
  - Other tests only compare `EndedAt` against the zero time, so they are unaffected.
- **Bar test (F4):** the clock check uses the last `\d\d:\d\d ` match. A mark rendered after the clock gives a mark index above that match and fails. The mark text has no `dd:dd` pattern. I could not confirm it passes, because it is tier2/tmux and not run this session.

**Calls made**

None. The context already held every changed file and the gate log excerpt.
