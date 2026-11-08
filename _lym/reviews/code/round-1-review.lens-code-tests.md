```yaml
lens: code-tests
outcome: reported
```

**Per-behaviour list (behaviour → pinning test)**

1. `ModuleMainPackages` lists main packages sorted, with `.` for the root, and skips library and testdata packages. It is pinned by `TestModuleMainPackages`, first subtest (`internal/impactset/mains_integration_test.go`).
2. `ModuleMainPackages` on a module with no main package returns an empty list and no error. It is pinned by the second subtest of `TestModuleMainPackages`.
3. `ModuleMainPackages` on a directory with no `go.mod` returns an error naming the directory. It is pinned by the third subtest of `TestModuleMainPackages`.
4. `ModuleMainPackages` keeps a package that fails to build, which is the point of `-e`. It has no test. The "dir == '' → list error" branch and the "directory outside the module" branch have none either. See F2.
5. `lym gate test --main-packages` without `--card` is refused, naming `--card <id>`. It is pinned by the new row in `TestGateTest_RefusesBeforeResolvingTheHub` (`internal/gatecli/cli_test.go`).
6. `--card` with `--main-packages` adds the module's main packages to the card's packages. It is pinned by the new subtest in `TestGateTest_Scopes` (`internal/gatecli/scopes_integration_test.go`). That subtest covers only the case where a module is already named and the main package is not.
7. `withMainPackages` (`internal/gatecli/scope.go`):
   - It does not add a main package a card already names (the dedupe branch). No test pins it. See F3.
   - It appends a module the cards leave out (the `index < 0` branch). No test pins it. See F3.
   - It leaves the list unchanged when there are no mains. No test pins it. See F3.
8. A failed `go list` refuses the call, naming the module and the way forward (`scope.go:201-204`). It has no test. See F3.
9. Non-go scopers (`none`, `dotnet`) add nothing. It has no test. See F3.
10. The detached runner's call words carry `--main-packages` (`internal/gatecli/detachfront.go:360`). No test in `detachfront_test.go` or `detachfront_integration_test.go` references `words()` or the flag. See F1.
11. `AgentGateCallWithMainPackages` renders the call with `--main-packages` last, and refuses an unknown key.
    - The rendering is pinned indirectly by `TestRenameBoundary_BatchesCheckAndGateTheirOwnTree` (`cardgate_test.go:145`).
    - The error path is pinned by the dangling-list subtest, which now runs with `last` true and false.
    - No direct test sits in `internal/gateslot/config_test.go` beside `AgentGateCall`'s. This is covered, so not a finding.
12. `isLastBatch` is pinned by the same rename-boundary test. Batch 1 gets no flag and batch 2 does.
13. `recoverSpawn` passes the last-batch flag. It is pinned by `recoverbatch_test.go:1741`.
14. `BeginBatch` passes the last-batch flag (`beginbatch.go:562`). The test at `beginbatch_test.go:420` does not pin it. See F4.
15. The stencil names `--main-packages`. It is pinned by the new `govalues_test.go` row. The `//lym:watch internal/gateslot/*.go` was added to match.

**Findings**

- **F1, MEDIUM, scope** — `internal/gatecli/detachfront.go:360`. The detached runner's `words()` appends `--main-packages`, and nothing asserts it. If the flag were dropped, a capped call and its runner would disagree on the scope, and every test would still pass. Add a case to the existing detach-front test of `words()` or of the runner's command line. I found no `words()` assertion in `internal/gatecli/*_test.go`.
- **F2, LOW, scope** — `internal/impactset/mains.go:50-69`. Two behaviours have no test row:
  - `-e` keeps a package that fails to build listed under its name. This is the stated reason for `-e`.
  - The error branches for an empty directory and for a directory outside the module are untested.

  Add a subtest with a package main that has a type error and assert it is still listed.
- **F3, LOW, scope** — `internal/gatecli/scope.go:200-206` and `283-297`.
  - Only the "module named, main added" path is tested.
  - Add rows to the `--main-packages` subtest (or a table) for a card that already names the main package (listed once), for a card in another module (the append branch), and for a `go list` failure (refusal text).
  - Add a row showing that `--main-packages` under the `none` scoper adds nothing. The Sketch and the docs promise this.
- **F4, MEDIUM, scope** — `internal/websterengine/beginbatch.go:562`. `BeginBatch` wiring of `isLastBatch` is unpinned. The begin test at `beginbatch_test.go:420` asserts the substring "- `lym gate test --gate batch --card 1-json-flag`", with no closing backtick. It passes with or without `--main-packages`. Dropping the flag from `BeginBatch` fails no test, though begin-batch is the main path. Add assertions on the begin prompt for a last batch (flag present) and a non-last batch (flag absent, via `wantNot`), as the recover test does for the last batch.

**Tool calls**
1. grep for "--gate batch" and `words()`/`detachfront` in test files. The manifest did not show which tests pin the batch-gate prompt or the runner words.
2. Read `beginbatch_test.go:385-425`. I needed to see whether the begin-batch assertion pins the new flag.
