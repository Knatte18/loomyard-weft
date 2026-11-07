# Gate: `{test_targets}` starts only binaries with a test of the tier

A tier's command can now pass its packages to `go test` through a new `{test_targets}` placeholder.
`lym gate test` fills it with each invocation's targets, narrowed to the packages that hold a test function the tier runs.
The tier of each test comes from the `tiergate.Require(t, N)` call that opens it.
A tier run then compiles, links and starts no test binary that would only answer `no tests to run`.
The tier 1 research report measured that at 130 of the 196 binaries a tier 1 pass starts,
and about 8 s of the 20.5 s floor on neve.
On a machine whose scanner holds each new binary, the saving should be larger.
`{targets}` keeps its meaning, so the `go build` and `go vet` steps of a tier command still cover the whole scope.

## What changes

- `internal/impactset`: `HoldsTierTest(dir, tiers)` reports whether a package directory holds a test function a run of those tiers executes.
  A top-level `Test`, `Benchmark` or `Fuzz` function (not `TestMain`) that opens with `tiergate.Require(t, N)` counts for tier N alone.
  Any other test function counts for every tier, because nothing skips it.
  A directory or test file that cannot be read or parsed counts as holding a test, so the test run reports the problem.
- `internal/gatecli`: `runInvocations` narrows each invocation when the tier's command holds `{test_targets}`.
  It expands pattern targets with the same `go list` the green record uses (`expandPackages`, now a plain function).
  It then keeps the packages `HoldsTierTest` accepts for the tier and the tiers it `covers`, plus any import-path target.
  An invocation left with no package runs no command and prints a line saying so.
  If listing fails, or the narrowed list exceeds `greenCommandBudget`, every target runs as before.
  `renderTierCommand` fills the new placeholder.
- `internal/gateslot`: `TestTargetsPlaceholder` and `HoldsTargets` are new.
  Every check for "the command takes targets" now goes through `HoldsTargets`: `all_targets` validation, scope invocations, the guard-step check and the green record.
  So a command holding only `{test_targets}` narrows by scope and records green the same way a `{targets}` command does.
- Docs: the `gatecli`, `gateslot` and `impactset` package docs and the gate template's comment describe the placeholder.
  The `gatecli` example now shows the hub's current `LYM_TEST_TIER` tiers using `{test_targets}`.
  `pattern/PATTERN-test-speed.md` gains a bullet for the rule.

## Design decisions

- **A placeholder, not a change to `{targets}`.**
  Narrowing `{targets}` would also narrow tier 1's `go vet {targets}`.
  That would drop full vet coverage of the 130 packages with no tier 1 test.
  With a separate placeholder, the command chooses which step gets the narrowed list.
  Hubs that don't use the placeholder see no change.
- **The list is derived each run, so it needs no hand edit.**
  A package gains a tier as soon as its first `Require` of that tier lands, and no package list is kept anywhere.
- **Any failure runs everything.**
  The narrowing only saves time, so a listing error, an unparsable file or a list that would make the command too long falls back to the full targets.
- **Applies to every tier**, as the entry asks, since each tier counts its own number and its `covers`.

## Hub config follow-up

The hub's `gate.yaml` still uses `{targets}` in every tier command, so nothing gets faster until it switches.
This change does not touch the hub file.
After landing, the prime should set each tier's `go test` targets to `{test_targets}`, for example:

```
lym config gate --set 'tiers.tier1.command=go build ./... && go vet {targets} && go test -json {test_targets} {args}'
```

Do the same for `tier0`, `tier2` and `tier3`, with `go test -json {test_targets} {args}`.
An older `lym` binary leaves the placeholder unrendered, so make the switch only once the landed binary is in use.

## Bounds

- A tier whose `LYM_TEST_TIER` names a range such as `0-2` runs tests of tiers it does not list in `covers`.
  A package that holds only those tests is left out.
  The hub's tiers each select a single tier, so this does not apply today.
- Build constraints are not read, so a test file that only another platform compiles still keeps its package in the list.
- An invocation that narrows to nothing also skips the command's other steps, such as `go build` and `go vet`.
  The same already holds for an empty scope, and the whole-scope landing gates still run them.

## Tests

- `TestHoldsTierTest` (tier0): a matching, other and missing `Require`; benchmark and fuzz functions; `TestMain`, a lookalike helper and a method; a non-test file; an unparsable file and an unreadable directory.
- `TestTierTestTargets` (tier0): a dropped package, a covered tier, an empty result, a kept import path and the command-budget fallback.
- New rows in `TestRenderTierCommand`, `TestPlanTierRuns` and the gate config refusal table cover filling the placeholder, scope narrowing for a `{test_targets}`-only command, and its `all_targets` requirement.
