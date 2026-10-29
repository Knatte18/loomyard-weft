```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: no findings.

**Per change, the set found:**

1. **`uncheckableRefusal` setup stores the record on `fx.Seeded` instead of a closure variable.**
   - Every caller of a setup from this factory passes the row's own `fx`: `fabricRefusalSetup`, `pauseRefusalSetup`, `notReadOnlySetup`, `nilClassifierSetup`, `mixedSetup`, `withPathSetup`, `longAloneSetup` (`recoverbatch_test.go:1013-1021`, rows at 1415-1534).
   - `atStartBaseSetup` runs through `atStartSetup` (1074-1078), which four rows call (1432, 1459, 1477, 1498).
   - `committedBaseSetup` runs through `committedSetup` (1079-1084), which two rows call (1442, 1513).
   - `longBesidePathSetup` builds a fresh setup and base check per invocation (1025-1030) and is used by one row (1422).
   - Each subtest runs `t.Parallel()` and builds its own fixture with `newRecoverFixture(t)` (1725-1727), so a row's `fx.Seeded` is written and read only inside that row. Covered.

2. **The check reads `rec := fx.Seeded` (`recoverbatch_test.go:1003`).**
   - Checks that call a base check: `fabricRefusalCheck` via `noStartCheck` (1072), `pauseRefusalCheck` (1085), `noStartCheck(longAloneBase)` (1428), `longBesidePathCheck` (1042), and the `atStartBase` row (1432-1438). All receive the same `fx` that the row's setup seeded. Covered.
   - Checks that call no base check: the `notReadOnly`, `nilClassifier`, `mixed` and `withPath` rows use their checks directly with their own `fx`. Covered.
   - The two `committedSetup` rows never call a base check. They discard it (`committedBaseSetup, _`), and no check reads `fx.Seeded` on those rows. Covered.

3. **New `Seeded` field on `recoverFixture` (`recoverbatch_helper_test.go:55`).**
   - `recoverFixture{` is built only in `newRecoverFixture` (`recoverbatch_helper_test.go:130`), which leaves the field nil.
   - The other `newRecoverFixture` callers (e.g. `StartFailures`, line 1882) never read it.
   - The field's doc comment sits on its own line above the field, begins with the field name, and has no end-of-line comment, so it follows the repo's convention. Covered.

4. **Docs, PATTERN and stencils.** The diff touches two test files only.
   - CLAUDE.md's "Docs land in the same commit" rule is not triggered: no new module, CLI behaviour or infrastructure.
   - Neither `PATTERN-test-speed` nor `PATTERN-test-isolation` is touched or violated: rows keep `t.Parallel`, there is no new test, and the diff stays inside `internal/websterengine` test files.
   - No PATTERN entries are declared as changed.
   - No stencils are changed.

5. **Commit message.** It carries no `Pattern-Departure:` trailer, and none is needed.

**Noted outside my lens, no finding:** the other closure variables (`longBesidePathBase`, `scratchPath`) are written by `longBesidePathSetup` and read by its check, but only one row uses them. The decision record puts this out of scope.

**Tool calls:**
1. `awk` over `recoverbatch_test.go` lines 1700-1903, to confirm each subtest builds its own `newRecoverFixture` and runs in parallel. The runner loop was not in my context.
