# Fix flaky TestRecoverSpawnOrAttach "record changed" in websterengine

`TestRecoverSpawnOrAttach` in `internal/websterengine` could fail with "record changed" over a record that had not changed, and passed on rerun.
The `uncheckableRefusal` setup-and-check pair shared one `rec` variable through a closure.
Rows that reuse a pair (four for `atStartSetup`, two for `committedSetup`) run as parallel subtests and each reassigned `rec` in its setup.
A row's check could therefore compare its fixture's record against another row's, which is also a data race on the variable.

The setup now stores the record it seeds on the subtest's `recoverFixture` (new `Seeded` field in `recoverbatch_helper_test.go`).
The check reads `fx.Seeded`, and the closure variable is gone.
Only test files change; no `internal/websterengine` production code is touched.

Closes #902

## Design decisions

- The fixture is built per subtest, so each row compares against the record its own setup seeded.
- Rows keep `t.Parallel`, and the refusal assertions are unchanged, including the pointer-identity check that a refused spawn leaves the same record in place.

## To check by hand

- Repeated runs of `TestRecoverSpawnOrAttach` under the race detector were not run here; run them to confirm the flake and the race are gone.
