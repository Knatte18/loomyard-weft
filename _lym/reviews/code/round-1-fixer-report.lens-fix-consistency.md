```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the fix's agreement with its surroundings.

1. **LOW**: `internal/planparser/latercaller.go`, the new `testSupportDirectories`.
   - It is a second hand-kept copy of the test-support set, and nothing on either side ties the copies together.
   - The set is declared in `testSupportPackages` at `cmd/lym/testonlycode_linux_test.go:35` and stated in `PATTERN-test-only-code` ("The test-support packages are `internal/testkit` with its kits, `internal/gitkit` and `internal/hubforge`").
   - The new comment says "the reachability guard counts as reached", but neither the guard's list nor the pattern's Enforcement section mentions the planparser copy. A package added to the guard would leave `create-caller-later` refusing functions the guard accepts.
   - The spec bullet in `loom-plan-checks-spec.md` and `_lym/landing/summary.md` each spell the three directories out again.

2. **LOW**: `_lym/landing/summary.md`, "A function is test code, and skipped, when any of these holds", against the refs-not-count bullet above it.
   - The list now includes "its unit is `internal/testkit`, …", so it calls testkit functions test code.
   - The bullet says "a later card's own new non-test `func` does count". In the code, a testkit `func` the later card creates counts as a possible caller, because `cardCanHoldProductionCaller` excludes only `testing.` heads, `_test` units and test-prefixed names.
   - The spec has the same ambiguity. It says testkit functions are "skipped", then "it is one, unless it is test code by its unit, its name or a `testing.` type", where "unit" means only `<dir>_test`.

The other siblings I checked agree with the change: the card-granularity rule at `loom-plan-spec.md` line 364, the edited Plan vs. schedule sentence, the doc comments in `latercaller.go` and the test's header comment.

Tool calls:
1. grep for `create-caller-later`, `testSupportPackages` and the PATTERN lines: my context lacked the references outside the files already read.
2. Read `loom-plan-spec.md` lines 352–372 and `PATTERN-test-only-code.md`, and grep for "first caller" in markdown: my context held neither text.
