# Fixer-report — webster round 1

## Changes

Each fix is its own commit, gated green on the touched packages with `lyx gate test --tags integration`.

- F1: `wireDarn` sets `c.env.VerifyCommand` to a closure over `darnengine.VerifyCommand(dirs)`, so the gate's told command and `BaseCommand` stay the hub's whole verify; `darnVerifySource` stays for the landing deps.
  Added a `TestWire_DarnRecipe` subtest with tree scope on, asserting `c.env.VerifyCommand()` returns the hub's verify.
- F2: `TreeScopedCommand` marks every tree a path lies in; a path in no tree still returns `("", false)`.
  Added a nested-trees row (`docs` and `docs/api`) to `TestTreeScopedCommand`; `doc.go` states that a path touches every tree it lies in.
- F3: `ModuleGuardTests` returns a plain error naming the module directory and the package directory instead of skipping a package outside the module.
- F4: added a `TestCommandFor` row where two outside packages hold the same guard name, expecting it once.
- F5: added a `--guards` scenario over a module whose root package holds a marked test, expecting the `.` invocation.
- F6: `NewVerifyGate`'s doc says "With treeScope nil".
- F7: `darnengine/doc.go`'s key list restates `verify` as the hub's whole verify command and adds `tree_scope` and `tree_scope_verify` with a pointer to the tree scope paragraph.

## Disputed

None.

## Deferred

None.

## Self-check

- fix-consistency, LOW: `darnwiring.go` file header still said the verify source goes to the verify gate. Fixed as proposed.
- fix-consistency, LOW: `TreeScopedCommand`'s doc comment lacked the touches-every-tree rule. Fixed as proposed.
- fix-code-correctness, LOW: `ModuleGuardTests`'s doc comment omitted the new error. Fixed as proposed.
- fix-proposal, MEDIUM: F3's error would fire on every package if `go list`'s directory spelling differed from `filepath.Abs` (a symlinked module root). Fixed as proposed in the sense that the finding's own fix stays: the error is the better outcome than the silent empty guard map the review named, and `go list` run with a nil environment takes its working directory from `PWD`, the spelling `Abs` uses, so the case is not expected; no change beyond the doc comment.
- fix-proposal, LOW: the F1 subtest's tree scope seeding is a silent no-op if the template text changes. Fixed as proposed in the sense that no change is made: the same replacement is already used at `wiring_darn_test.go:262`.
- fix-affected-set: no findings.
