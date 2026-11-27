```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. For each function or branch the diff adds or changes, run one concrete case and report what is wrong.

**Cases run**

- `withOnlyTests`:
  - no `--gate`: returns unchanged. Correct.
  - scoper `none`: returns unchanged. Correct.
  - no `go.mod`: holds nothing and leaves `marked` nil, so a marked key then runs nothing (finding 2).
  - `go list` fails on a diff key: warns and holds nothing. Correct as documented.
  - `go list` fails on a marked key: refuses. Correct.
  - a marker naming a key `gate.gates` lacks: refuses. Correct.
  - two keys on one test: held or marked correctly. Correct.
  - a stalled `go list`: no context bounds it (finding 4).
- `withOnlyTests` with a filter flag after `--`: `held` still fills, and `withoutHeld` still drops the held guards from the guard step (finding 5).
- `markedScope`:
  - scoper `none`: refuses. Correct.
  - marked tests in tier 1 with key tiers `[0]`: prints a line, runs nothing, exits 0 (finding 2).
  - plain call with key tiers `[0, 1]`: the guard step runs exactly the marked tests. Correct.
  - `--no-carry`, or a tier set to `carry: false`: the listed tier runs whole (finding 1).
- `markedNames` with an empty map: nil. Correct.
- `withoutHeld`:
  - nil guards: stay nil. Correct.
  - empty `held`: returns guards unchanged. Correct.
  - a held name in one directory: dropped there, the rest kept. Correct.
  - collisions: matching is by bare name, but no held name occurs twice in the repo today, so it is a bound only. Correct today.
- `printHeldLine`: formatting only. Correct.
- `isMarkedCall`:
  - explicit targets: false. Correct.
  - no `--gate`: false. Correct.
- `skippedInside`:
  - nil `inside`: empty map. Correct.
  - a tier-1 guard inside a tier-0 group's set: kept. Correct.
  - a tier-0 guard when tier 1 covers 0: dropped. Correct.
  - two base groups: a guard may also run in the other group's listed run. That is a duplicate run, not a miss. Correct.
- `computeScope`:
  - `anyWhole` case: `scanGuards` with empty `listedTiers` counts every guard as left out. Correct.
  - narrowed case: merges `skippedInside`. Correct.
- `planTierRuns` skip union:
  - only watched: same as before. Correct.
  - only held on a whole run: skip added, recorded args empty. Correct.
  - watched plus held: recorded args keep only the watch skip. Correct.
  - filter flag: no skip and `run.held` nil. Correct.
  - zero invocations: no skip. Correct.
- `greenArgs`: with `held` empty returns `args`, otherwise `unheldArgs`. Correct.
- `greenRecord.with` `sameKey` now includes `Held`: an old entry and a new one sit side by side. Correct.
- `covered` and `heldWithin`:
  - a Parent-Align entry with the upkeep tests held, against a Finalize request holding the same set: covered. Correct.
  - a webster entry holding the landing tests, against Finalize: not covered. Correct.
  - a parent-align entry against an upkeep guard-step request (`held` nil): not covered. Correct.
  - a plain `--all` entry against upkeep: covered, which is correct because that run ran the marked tests.
- `newestWholeAncestor` with held, in both `diffBaseGroups` and `carriesOverInert`: an entry with a subset of held counts, one with extra held does not. Correct.
- `recordInertCarried` and `recordPass` store `Held: run.held`; a guard step's `Held` is nil, so its entries match only on their exact `-run` args. Correct, conservative.
- `targetsFor` and `InsideGuards`:
  - an unwatched guard on either side of the set: dropped. Correct.
  - no guard inside the set: `InsideGuards` stays nil. Correct.
- `modulePackageDirs` refactor: same errors as before. Correct.
- `FindOnlyTests` and `onlyTestsInFile`:
  - marker with no key: `OnlyMalformed`. Correct.
  - marker followed by prose: misplaced. Correct.
  - marker on a method: misplaced. Correct.
  - `//lym:onlyx`: ignored. Correct.
  - parse error: `GuardParse`. Correct.
  - dir `.`: path handled. Correct.
- `tidyUpkeepStep`:
  - empty call: no-upkeep fragment. Correct.
  - with a call: marker filled, and `Fill` strips the leading comment. Correct.
- `daemon.upkeepCall`:
  - nil dep: empty. Correct.
  - render error: logged, empty. Correct.
- `hubDobbyDepsStarting` `UpkeepCall`: on a prime of scoper `none` or `dotnet` it renders a call that is always refused (finding 3).
- `decodeGates` with `marked`: accepted, and the message is updated. Correct.

**Findings**

1. MEDIUM, design — `internal/gatecli/tiers.go` `scopeInvocations` / `internal/gatecli/only.go` `markedScope`. A marked key's listed tiers run their whole suite under `--no-carry`, or when a listed tier is set to `carry: false`.
   - Evidence: `planTierRuns` sets `tier.NoCarry = tier.NoCarry || request.noCarry` (tiers.go, near the start of `planTierRuns`). `scopeInvocations` then hits `case sc.whole, tier.NoCarry: return runsWhole` before the `len(sc.modules) == 0` case.
   - Result: `lym gate test --gate upkeep --no-carry` runs tier0 over `./...`, skipping only the held tests. It should run only the marked tests.
   - Fix: `markedScope` must plan no listed invocation under any flag, for example by a `scopeMarked` case placed first in `scopeInvocations`.
2. MEDIUM, design — `internal/gatecli/only.go` `markedScope` and `internal/gatecli/run.go`, after `planTierRuns`. An upkeep call that runs none of its marked tests exits 0, so it reads as a clean upkeep. This happens in three cases:
   - every marked test sits in a tier the key does not list: this hub, on the template's `tiers: [0]`, with every upkeep test in tier 1;
   - the scanned directory has no `go.mod`;
   - no test is marked.

   Evidence: `markedScope` only prints a stderr line before `continue`, and `run.go` only does `logger.Info("... no guard test is left for the guard step; it runs nothing")`. The board-tidy agent files notes only for failures, so every upkeep run passes silently until `gates.upkeep` is changed.
3. MEDIUM, design — `internal/beamcli/dobbydeps.go`, the new `geom.Slug == ""` block. `UpkeepCall` renders `AgentGateCall(UpkeepGateKey)` whatever the scoper. On the template default, `scoper: none` (template.yaml), or under `dotnet`, `markedScope` always refuses ("needs the go scoper"). So on every non-Go hub the prime's tidy agent runs a refused call every `board_tidy_every` landings. The stencil has it file only test failures, so nothing tells the operator.
   - Fix: return an empty call when `cfg.Scoper != "go"`.
4. LOW, design — `internal/gatecli/only.go` `withOnlyTests` → `internal/impactset/guards.go` `modulePackageDirs`. It now runs before `computeScope` on every `--gate` call, including the scope-all keys `parent-align`, `finalize` and `webster`.
   - Its `go list` is `exec.Command` with no context, and `withOnlyTests` receives neither `preCtx` nor `ctx`. A stalled `go list` hangs the call past the pre-tier ceiling, which `computeScope`'s doc says bounds `go list`.
   - It also lists the module a second time on calls that later run `scanGuards`.
5. LOW, design — `internal/gatecli/run.go`, the `withoutHeld` call, against `doc.go` and the change description. Both say "a call whose words after `--` carry a filter flag … hold nothing back".
   - Under a filter, `planTierRuns` adds no skip, but `withOnlyTests` still fills `held`, and `sc.guards = withoutHeld(sc.guards, request.held)` still drops the held guard tests from the guard step.
   - Example: `--gate card --card 1 -- -run X` leaves out of the guard step the held guards it used to run. That contradicts the documented contract.

**Tool calls**

1. Grep for name collisions of the `//lym:only` tests, `isFilterFlag`, `onlyDirectiveLinesAfter` and `stencil.Fill`'s comment stripping: my context lacked these definitions and the repo-wide name check.
