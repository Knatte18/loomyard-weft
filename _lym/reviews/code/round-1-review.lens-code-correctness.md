```yaml
lens: code-correctness
outcome: reported
```

**List (function or branch, case run, verdict)**

- `HoldsTierTest`, an external `package p_test` file next to the package's own tests: both are `_test.go` files in the same directory and are read together. Correct.
- `HoldsTierTest`, a helper-based `Require` or a `Require` that is not the first statement: `RequiredTier` returns `required=false`, so the function counts as a test for every tier. The package is kept, never wrongly dropped. Correct.
- `HoldsTierTest`, a file carrying a build constraint: constraints are not read, so the package is kept. Correct and conservative.
- `HoldsTierTest`, an unreadable directory or an unparsable test file: returns true. Correct.
- `HoldsTierTest`, a package with only `Testable`, a method named `TestA`, or `TestMain`: dropped. Correct, since `go test` runs none of them.
- `HoldsTierTest`, a package with only `ExampleXxx` functions in a `_test.go` file: dropped, which is wrong. See F1.
- `isTestFunctionName`, `Test`, `Test_x` and `TestÄ`: kept. `Testx` and `Testable`: dropped. Matches Go's rule.
- `tierTestTargets`, target `./...` and a root package `.`: `expandPackages` runs `go list` and returns `./<rel>` or `.`, which pass the `HasPrefix(".")` check. `filepath.Join(moduleDir, ".")` resolves correctly. Correct.
- `tierTestTargets`, a plain target such as `./a`: it is cleaned by `normalizePackage` (`internal/gatecli/green.go`, `normalizePackage`), keeps its `./` prefix and is checked. An import path does not start with ".", so it is kept without a check (documented, and pinned by the test row). Correct.
- `tierTestTargets`, `go list` fails: logs a warning and returns the full `targets`. Correct.
- `tierTestTargets`, narrowed list longer than `greenCommandBudget`: returns the full `targets`. Correct.
- `tierTestTargets`, `slices.DeleteFunc` on `packages`: `expandPackages` returns a fresh slice from `unionSorted`, so the caller's `targets` is not mutated. Correct.
- `runInvocations`, narrowing yields an empty list: prints a line, `continue`, and runs no command. Not appended to `commands`. The exit is 0 if every invocation is skipped. Correct locally. I did not trace how the counter and green record treat a pass with zero commands, because `internal/gatecli/green.go` `recordPass` still receives these invocations in `ran`. Recording them as green is semantically fine, since they hold no test of the tier.
- `runInvocations`, command without `{test_targets}`: `testTargets` stays `invocation.targets`, so nothing changes. Correct.
- `renderTierCommand`, `testTargets` nil: `quoteAll(nil)` gives an empty string. Not reachable when the placeholder is present. Correct.
- `gateslot.HoldsTargets` and the `decodeTier` check: a command with only `{test_targets}` requires `all_targets`. Correct. The swap in `green.go` and `tiers.go` is consistent at every site I saw.
- `expandPackages`, now a free function: passes `goBinary` and `resolveRunDir(g.root, …)` at all four call sites, equivalent to the old method. Correct.

**Findings**

1. LOW, design, `internal/impactset/tiertests.go:516-529` (`isTestFunctionName`), evidence `internal/impactset/tiertests.go:481`. `testFunctionPrefixes` omits `Example`. `go test` runs `ExampleXxx` functions that carry an `// Output:` comment, so a package whose only tests are examples is dropped from `{test_targets}` and its examples stop running. The goal's "no test lost" is therefore not guaranteed. My grep found no `func Example` in `internal` or `cmd` today, so it is latent. The fix is to add `Example` to the prefixes. An example declares no `testing.TB`, so it counts for every tier. Add a test row.

Tool calls (3):
1. Read `tiergate.Require`, grepped `^func Example` in `internal` and `cmd`, and read the tail of `expandPackages` and `normalizePackage`. My context lacked the `Require` semantics, whether examples exist, and the output form of the package list.
2. Grepped `internal/gatecli/tierrun.go` and `internal/gatecli/green.go` for a "no tests ran" check and for how skipped invocations reach `recordPass`. This was to settle the empty case. It showed no zero-test refusal in `tierrun.go`, and `recordPass` takes all `ran` invocations.
