```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, which pairs each behaviour the diff adds or changes with the test row that pins it.

**Behaviours and the rows that pin them**
1. `--gate card` lists only the cards' own packages at tier0: pinned by `TestGateTest_Scopes` / "--gate card runs the cards' packages alone with no guard step, says so and records the key" (`internal/gatecli/scopes_integration_test.go:826-141`, args `./lib` and `.`, no `./user`).
2. `--gate card` lists only the cards' own packages at every tier above tier0: pinned by the batch-versus-card row (`scopes_integration_test.go:108`, two tiers of `./lib` and `.`).
3. `--gate card` plans no guard step: pinned by the same two rows, since the `wantArgs` lists carry no `guardInvocation` (`:108`, `:132`).
4. `--gate card` prints the stderr line naming `--gate batch`: pinned (`:136-138`).
5. `--gate card` skips `go list` for dependents: pinned indirectly, because the args in rows 1 and 2 would gain `./user` if `cardDependents` ran.
6. `--card` with no key, or under `--gate batch`, keeps the dependents and the guard step: pinned by `:530` (`--card 1 --tier 1`, `guardInvocation`), `:111` and `:116-118` (batch row with `./user` and `guardInvocation`).
7. The held guard stays out of the guard step: the row moved from `--gate card` to `--gate batch` (`:116`), the key that still has a guard step. The held `-skip` on the listed run is key-agnostic and stays pinned through the upkeep row (`:120`). Nothing was loosened.
8. The vet step still runs under `--gate card` over the cards' own packages: **no row**. The vet rows (`:741-746`, `:756-783`, `:1074`) assert `./lib ./user` and `guardInvocation`, so they run `--card` without the card key.
9. `--guards` forces a guard step under `--gate card`: **no row**. Every `--guards` test drives explicit targets (`internal/gatecli/test_integration_test.go:1161`, `:2451-2461`, `:2501`, `:2518`, `:2589`).
10. Renaming `CardGateKey` and dropping the duplicate constants: refactor only. Existing config and priority tests cover the unchanged behaviour, and the build is the gate's to check.
11. The fixer prompt, the Webster and Weaver stencils, the spec, `doc.go`, the help text, the template comment and the PATTERN bullet: static prose, exempt.

**Findings**
- LOW, class `design`. Location: `internal/gatecli/scope.go:245-257` (vet step under `ownOnly`). No test pins that a `--gate card` call still plans the `go vet -tags tierN` run over the cards' own packages. `_lym/landing/summary.md:17` states "The vet step over the cards' packages stays". The only vet rows run `--card` without the key (`internal/gatecli/scopes_integration_test.go:741-746`, args `./lib ./user` plus `guardInvocation`). The fix is a `--gate card` assertion in the vet row at `:741`, expecting `./lib` and `.` plus the two vet invocations and no guard run.
- LOW, class `design`. Location: `internal/gatecli/scope.go:257` together with `run.go:122` and `tiers.go:206` (`--guards` forced while `guardStep` is false). No test pins that `--gate card --card &lt;id&gt; --guards` still runs a guard step over the guard tests the own-packages-only listed runs leave out. `_lym/landing/summary.md:16` states "`--guards` still forces one". Every `--guards` test uses explicit targets (`internal/gatecli/test_integration_test.go:1161`, `:2451-2461`). The fix is an assertion in the `--gate card` row at `scopes_integration_test.go:826`, run with `--guards`, expecting `guardInvocation` after the listed runs.

Tool calls:
1. A grep of `internal/gatecli/*_test.go` for `--guards`, of `scopes_integration_test.go` for vet rows, and of `carddependents_integration_test.go` for its call form. The manifest held none of those test rows.
