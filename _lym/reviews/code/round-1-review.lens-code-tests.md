```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, I looked for the test row that fails if it breaks.

**Per behaviour, the test that pins it**
- `dependentsFor` keeps the seeds first, adds transitive and test-import dependents sorted after them, keeps a seed that does not exist yet, and applies the `cmd/lym`→`lymbin` rule. Pinned by `TestDependentsFor` rows 1–4 (`internal/impactset/dependents_test.go:126-145`, graph at `derive_test.go:17-28`).
- `DependentPackages` runs `go list` under the tier tags (a tier1-tagged test import is added), leaves an unrelated package out, and errors with the directory's name. Pinned by `TestDependentPackages` (`dependents_integration_test.go:73-98`).
- `cardDependents` passes the result through on success. Pinned by `TestCardDependents` row 1 and, out of process, by the `TestGateTest_Scopes` row "--card runs a card's packages and the packages that import them…" (`scopes_integration_test.go:405`).
- `cardDependents` falls back to the cards' packages when `go list` fails. Pinned by `TestCardDependents` row 2 (`carddependents_integration_test.go:42-49`). Nothing asserts the Warn log or any operator-visible notice, so no test checks that the narrowing is reported.
- `computeScope`'s card branch takes the widened targets for the tier run, per module with nested-module cwds. Pinned by `TestGateTest_Scopes` rows at :405 (args and cwds :412), :569 (`--gate card`) and :541 (timeout).
- The vet step runs over the widened set. Pinned by `TestGateTest_Scopes` :490 (`vet -tags tier1 ./lib ./user`).
- `--main-packages` merges into the widened set without duplicates. Pinned by `TestGateTest_Scopes` :581.
- The guard step's `inside` set now holds the dependents, so a dependent's guard tests leave the guard step. **No test.** The fixture's `guardpkg` imports nothing (`scopes_integration_test.go:99`), so no row has a guard test in a package that joins the scope through the widening.
- The watched-test skip over the widened set (`FindWatchedTests` over the dependents' dirs). Partly pinned: row :456 has `watched` as a card package, but no row has a watched test in an added dependent.
- `--map --card` maps the dependents. **No test.** `TestGateTestMapInATaskWorktreeKeepsItsOwnMaps` (`testmap_integration_test.go:735`) has a single-package module with no dependents.
- A ctx-cancelled `go list` inside `cardDependents` narrows silently. **No test.**

**Findings**

1. MEDIUM · design · `internal/gatecli/scopes_integration_test.go:405`
   No test pins how the guard step treats a `//lym:guard` test in a package that joins the card scope only as a dependent. That covers both the case where its tier is listed (it runs in the tier run) and where it is not (it now runs nowhere).
   - Evidence: `scope.go:577-590` treats every target as inside. The fixture's guard package `guardpkg` (`scopes_integration_test.go:99`) imports no card package, so the widening never reaches it.
   - Fix: add a row whose guard package imports `lib` and holds a guard of an unlisted tier, asserting the guard still runs at the card gate.

2. LOW · design · `internal/gatecli/testmap.go:427-437`
   No test covers `--map --card` mapping the dependent packages, although the change description says the widening covers it.
   - Evidence: `testmap_integration_test.go:735` has a single-package fixture; `candidatePackages` reads `share.modules`, which now hold the dependents.

3. LOW · design · `internal/gatecli/scope.go:286-292`
   The fallback's only signal is `logger.Warn`, and no test checks that a narrowed card scope is visible to the caller. `TestCardDependents` row 2 asserts only the returned slice (`carddependents_integration_test.go:46`).

4. NIT · consistency · `internal/gatecli/carddependents_integration_test.go:28-40`
   Test economy: the success row of `TestCardDependents` asserts a passthrough of `DependentPackages`. `TestDependentPackages` row 1 (`dependents_integration_test.go:73-90`) already pins that over the same `lib`/`user` shape, and so does `TestGateTest_Scopes` :405 (`./lib ./user`).
   - Fix: fold the row into the failure row's test, or mark it `//testtiming:keep` naming the coverage-map visibility it buys.

Tool calls:
- Read the `TestGateTest_Scopes` fixture and its watch and guard rows: the manifest had only the changed hunks.
- Read `testGraph` and the guard coverage in `derive_integration_test.go`: `TestDependentsFor` uses that shared graph, which the manifest lacked.
