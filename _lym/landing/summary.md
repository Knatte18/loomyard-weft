# Remove code no path from any main reaches

Dead-code analysis over `cmd/lym` and the programs under `tools/` found one production function that no `main` reaches, and a set of test helpers no test calls.
This change deletes them and the tests that covered only them.
Behaviour is unchanged, and nothing a user can call is removed.

## What changes

- `internal/runfsm`: `Order` is removed, with its `OrderListsEveryStateHighestFirst` subtest and the `testOnlyCodeAllowed` entry in `cmd/lym/testonlycode_linux_test.go` that kept it alive for the test-only-code scan.
  The `Derive` doc comment now describes its precedence directly instead of pointing at `Order`.
  The unexported `order` table stays, since `Derive` reads it.
- Test helpers no test calls are removed from `battenshed`, `burlerengine`, `fabricengine`, `loomshed`, `orchcli`, `reedengine`, `reportengine`, `shedadapters` and `websterengine` test files.
  In `burlerengine`, this includes the fake shuttle's Stop-call log and the `stopCalls`, `stop`, `blocked` and `settled` helpers.
- `shuttleengine/startup_test.go`: the one-line wrapper `skillStartFixture` is removed and its doc comment folded into `skillStartFixtureWith`.

The diff is 5 insertions and 195 deletions over 15 files.

## Design decisions

- `Order` was dead only in production: no `main` reached it, and its sole caller was its own test.
  Its allowlist entry said it would go once the board displays run state from the FSM; with no such caller, it is deleted rather than kept for that plan.
- Test helpers are removed only where no test in the package calls them.
  A hit reached through reflection, a registry, `go:linkname` or a build tag for another platform was to stay; the diff touches no such code.

## To check by hand

- The dead-code report itself was not attached to this change, so the reviewer may want to rerun `golang.org/x/tools/cmd/deadcode`, with and without `-test`, over `cmd/lym` and `tools/` to confirm no hit remains that should have gone.
- No regression check ran for this task.
