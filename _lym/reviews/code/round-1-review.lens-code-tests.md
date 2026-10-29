```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens — for each behaviour the diff changes, name the test row that pins it.

**Per-behaviour list**

1. **Each refusal row's check reads the record its own setup seeded (the data-race fix).**
   - The test files are the behaviour here, so no separate test exists.
   - Rows reusing a pair are `atStartSetup` (4 rows), `committedSetup` (2 rows), `longBesidePathSetup`, `fabricRefusalSetup` and `pauseRefusalSetup`.
   - `-race` runs of `TestRecoverSpawnOrAttach` would flag a regression. That is a run-level check, not a row.
   - The old failure was a nondeterministic interleaving, which a row cannot deterministically pin.
   - Not a finding.

2. **Pointer identity: `fx.Deps.State.Batches[1] != rec`, i.e. the refused spawn leaves the same record in place.**
   - Pinned by the `uncheckableRefusal` check (recoverbatch_test.go:1003-1005).
   - Every row whose check calls the base check hits it: fabricRefusal, pauseRefusal, notReadOnly, nilClassifier, mixed, withPath, longAlone, longBesidePath, and the at-start row (`atStartBase`).
   - Pinned. It would fail if the spawn replaced the record, since `rec` now comes from `fx.Seeded`, the pointer the setup stored.

3. **Terminal, Status and StrandGUID assertions on the refused record.**
   - Same line, same rows as item 2.
   - Pinned.

4. **`fx.Seeded` is set by the setup and read by the check on the same fixture.**
   - Every base-check caller runs a setup from `uncheckableRefusal` first, so `Seeded` is non-nil.
   - Caller paths checked:
     - Rows at 1417 and 1422 call `fabricRefusalSetup` and `longBesidePathSetup`.
     - `longBesidePathSetup` calls the inner `setup(fx)`.
     - The `atStartSetup` row (1432) runs `atStartBaseSetup`, which sets `Seeded`.
   - If `Seeded` were left unset, the check would nil-dereference `rec.Terminal` after the pointer comparison and fail loudly.
   - Pinned. No stale-value path exists, because the fixture is built per subtest.

5. **`committedBaseSetup`'s discarded check.**
   - Rows using `committedSetup` (1442, 1513) assert their own errors and never read `Seeded`.
   - No behaviour is lost, because the base check was never run for those rows.
   - No finding.

6. **The `Seeded` field on `recoverFixture` (recoverbatch_helper_test.go:54-55).**
   - It is a test-support field, and its use is covered by items 2-4.
   - Other `recoverFixture` constructions: `recoverbatch_helper_test.go:130` is the only `&recoverFixture{` literal I found, and it leaves `Seeded` nil.
   - Tests that don't call the refusal setup are unaffected.
   - No finding.

7. **No new test added.**
   - The diff adds no test function or row, so there is no test-economy finding.

**Findings:** none.

**Tool calls:** none. The manifest already held the diff, the rows at 1417-1532 and the fixture literal at helper_test.go:130.
