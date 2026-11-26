```yaml
lens: code-correctness
outcome: reported
```

Code-correctness lens: I ran each function and branch the diff adds or changes with a concrete input. One BLOCKING regression, one MEDIUM swallowed error and one LOW case.

**Per function or branch: the case I ran, and whether it is correct**
- `dependentsFor`, the seed `./lib` over a graph whose `byDir` holds `lib`: `path.Clean` gives `lib`, the package is seeded, and the result is `./lib` followed by the sorted dependents. Correct.
- `dependentsFor`, the seed `.` for the root package: it is seeded as `.`, and `packageArgs` spells it `.`. Correct.
- `dependentsFor`, a seed the module does not hold yet (`./internal/new`): no seed is added, the seed stays first and nothing is added for it. Correct, as documented.
- `dependentsFor`, an empty or nil packages list: the impacted set is empty and the result is nil. Correct; `CardPackages` never emits an empty module anyway.
- `dependentsFor`, a seed written unclean (`./lib/`): it is seeded through `Clean`, but `slices.Contains(result, "./lib")` is false against `./lib/`, so `./lib` is appended a second time. Not correct, finding F3.
- `dependentsFor`, with `cmd/lym` as a dependent rather than a seed: the lymbin rule does not fire, so packages whose tests run the lym binary are not added. This matches the doc and the diff scope. Correct as documented; none of the issue's packages (`burlercli`, `configcli`, `loomengine`, `shuttleengines`) has a test that imports lymbin.
- `DependentPackages` over a directory with no `go.mod`: `listPackages` fails and the error is wrapped with `moduleDir` in it. Correct.
- `DependentPackages` with a cancelled ctx: `go list` is killed and an error is returned. Correct; the caller handles it below.
- `cardDependents` when `go list` succeeds: it returns the dependents. Correct.
- `cardDependents` when `go list` fails, for example over a package whose import is not yet satisfiable: it calls `logger.Warn` and returns `module.Packages`. Nothing reaches `request.stderr`, so the gate can pass over the narrower set. Finding F2.
- `cardDependents` with a cancelled ctx: it narrows silently, and `preTierExpiry` in `run.go:112` then refuses. Correct in effect; the warning misreports the cause as a listing failure.
- `computeScope`'s card branch over the widened targets: the `os.Stat` filter on dirs, `FindWatchedTests` over more dirs, `tagTiers` over more dirs and `withMainPackages` dedup (a main that is already a dependent is not added twice) all work. Correct.
- `withMainPackages` when the targets are the planparser slice (the failure path): its `append` may write into the planparser backing array. That array is local to the call, so this is harmless. Correct.
- `scanGuards` with the widened `inside` set, for the card key (`tiers: [0]`, `internal/gateslot/template.yaml:35`, which the hub does not override): `cmd/lym` is inside for nearly every card, so its tier1 guards run nowhere. Not correct, finding F1.
- `--map --card` through `candidatePackages` over the widened modules: it maps more packages. Correct; that is the intended consequence.

**Findings**

- **F1 — BLOCKING, design**
  - Location: `internal/gatecli/scope.go:198`, which widens the scope, together with `scanGuards` at `internal/gatecli/scope.go:577-590`.
  - Summary: the widened card scope puts `cmd/lym` inside on almost every card. `scanGuards` drops every guard of an inside package from the guard step, whatever its tier. The `card` key lists tier0 alone, so `cmd/lym`'s tier1 guards, `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain` among them, no longer run at the card gate. Before the diff they ran in the guard step.
  - Evidence:
    - `go list -deps ./cmd/lym` lists 188 internal packages, including `planparser`, `shedrecipe`, `impactset` and `gatecli`, so `cmd/lym` reaches nearly any card package.
    - `cmd/lym/testonlycode_linux_test.go:79-81` marks that test `//lym:guard` with `tiergate.Require(t, 1)`; three of the `cmd/lym` guard tests are tier1.
    - `internal/gateslot/template.yaml:35` sets `card: {tiers: [0], ...}`.
    - `scope.go:588` keeps a guard only when `!inside[path.Clean(dir)]`.
    - `tiers.go:189-200` plans guard runs from `sc.guards` alone.
  - Effect: the same holds for a tier1 guard in any other dependent package, such as `internal/gatecli` or `internal/impactset`. The webster implementer stencil's claim that the guard step "can fail on … the pinned set in a `cmd/lym` test" no longer holds for those guards.
  - Fix: count a guard test as inside only when a listed tier, or a tier a listed tier covers, is its own tier.

- **F2 — MEDIUM, design**
  - Location: `internal/gatecli/scope.go:286-293` (`cardDependents`).
  - Summary: a failed listing of the dependents is only logged at Warn, and the call continues over the cards' packages alone. Nothing reaches `request.stderr`, so the caller cannot tell the gate passed over a narrower scope than documented.
  - Contrast:
    - The diff scope names its fallback on stderr (`scope.go:256`).
    - `--main-packages` refuses on the same `go list` failure (`scope.go:203-204`).
  - When it bites: the listing fails on any load error, because `listPackages` runs without `-e` (`derive.go:60`). A broken package outside the cards' packages, which the listed tiers then never compile, lets the card gate go green with every dependent dropped.
  - Fix: print the narrowing on `request.stderr` with its cause, or refuse as `--main-packages` does.

- **F3 — LOW, design**
  - Location: `internal/impactset/dependents.go:42-46`.
  - Summary: the seed is matched with `path.Clean(arg)`, but the de-duplication compares the raw `arg`. An unclean seed such as `./lib/` or `lib` is therefore listed twice: once raw, once as `./lib` from `packageArgs`.
  - Today's caller is safe, since `appendPackage` (`planparser/targetdirs.go:129-137`) spells every argument cleanly as `./dir`. The doc of the exported function does not require clean arguments.
  - Fix: normalise the seeds once and compare normalised values.

**Tool calls**
1. I grepped for `testkit/lymbin` imports in the tests of `burlercli`, `configcli`, `loomengine` and `shuttleengines`, and counted them repository-wide. My context lacked whether the lymbin rule's asymmetry hits the issue's packages.
