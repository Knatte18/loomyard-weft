# Support log — gate-scope-by-coverage-map

## Interview

Autonomous run: no operator answered; every pick below is an auto-pick recorded in the question ledger.

### Turn 1 — exploration

Two exploration subagents read the gate scope and the coverage maps.
Findings the design rests on:

- `computeScope` over `impactset.DeriveTargets` forms the diff scope: changed packages plus every importer, with whole-tree fallbacks, guard and watch tests.
- The hub's `gate.yaml` runs `darn-write` and `code-burler-fix` on tiers 0 to 2 with `scope: diff`; `finalize` runs everything.
- Maps hold only the test's own package's blocks (coverpkg is the package itself), so they cannot select tests of importing packages.
- testmap maps tiers 0 and 1 only; tier 2 is unmapped.
- `BuildBinary` already takes a coverpkg list; Go 1.26 supports multi-package coverpkg.

### Turn 2 — design

The own-package-only map forced D2: the board's idea needs cross-package records.
Strict freshness (D5 file-hash rule) was picked over a relaxed one, per the board's "widen, never narrow".
Review round 1 moved that rule from the tier's base to the map's own `Commit`.

### Turn 3 — lens review

Every fork returned a report.

- bound-breaker 1 (BLOCKING): testmap does not drop spawning tests. Fixed: D2 records the spawn verdict, D3 always runs such tests.
- bound-breaker 2 and walkthrough 1 (BLOCKING, merged): no source of block boundaries without a build. Fixed: D4 matches at function level from `go/parser`, with the block keys' enclosing function.
- existing-paths 1 (BLOCKING): dependency hashes in `Files` would make testmap rebuild every map. Fixed: D2 puts them in a field of their own; `stale` and `mapDrift` keep own-package `Files`.
- interactions 2 and existing-paths 2 (merged): dependency drift stays a miss until the age limit. Fixed: D2 adds covered-package drift to testmap's staleness; the cost is a question to the parent in Open risks.
  Reversed in round 3 by the parent's ruling.
- bound-breaker 3 and walkthrough 2 (merged): cross-package keys and hashes carry no package. Fixed in D2.
- walkthrough 3: a new file in a covered package passes the hash check. Fixed: D2 records the file set, D5 compares it.
- interactions 1: guard tests lost in narrowed packages. Fixed: D3 always runs a narrowed package's guard tests.
- interactions 4 and existing-paths 3 (merged): `tree_scope` and explicit targets. Fixed: D1 states they are never narrowed.
- walkthrough 4: the derivation's flat package list. Fixed: D3 carries the changed set apart, and binary-test packages run whole.
- walkthrough 5: unrecorded tests limited to the tier. Fixed in D3.
- existing-paths 4: the selected green entry. Fixed: D6 records it with its `-run` arguments.
- existing-paths 5: redundancy fresh build. Fixed: D2 keeps an own-package coverpkg there.
- existing-paths 6: `writeFailed` and the marker. Fixed in D2.
- existing-paths 7: template comment. Fixed: a `Review:` criterion.
- bound-breaker 5 and walkthrough 6 (merged): tier 1 builds twice. Fixed: D6 states the bound; the build cache serves the second.
- bound-breaker 6: overlong `-run`. Fixed: D6 widens past the command budget.
- bound-breaker 7 and walkthrough 7 (merged): torn reads cannot occur and `ReadMap` takes a lock. Fixed: D8 reads through `ReadMap`, a failed read is a miss.
- bound-breaker 4: drift in a package no test covered. Fixed: D5's bound reworded to the precise case.
- interactions 3: "exactly" against D6's overlap. Fixed in the D3 criterion.
- board-scope 1 (decision): D2 ahead of `testmap-upkeep`. Kept as a question for the parent in Open risks; the parent answered it in round 3 (Q10).
- board-scope 2 (decision): this hub's `gate.yaml`. Kept as a question for the parent in Open risks; moved to Scope Out in round 1.
- board-scope 3: sweep delayed by the rebuild. Fixed: one clause in Open risks.
- leanness 1 to 7: fixed: the `pr-review` assumption cut, the tier-2 assumption trimmed, the Scope Out bullet made bare,
  the `BuildBinary` hint moved to Notes, D7's last sentence cut, the test-speed constraint trimmed, the Scope In tier claim dropped.

## Rejected alternatives

- Change the meaning of `diff` instead of adding `coverage`: a meaning change needs a reinterpretation and would also narrow `pr-review`.
- Keep own-package maps and narrow only the changed packages: the cost is in importing packages, so it saves nothing.
- Narrow tier 2 to the changed packages without maps: narrows without a record, against the board's miss rule.
- Relaxed freshness, checking only the changed package's and the importer's own file hashes: misses a call path a third package gained since the map was built.
- Let the gate rebuild a stale map before scoping: holds slots and duplicates `testmap-upkeep`'s scope.
- Treat every new function as a miss: widens on nearly every diff, while a plain function is reached only through a changed caller.
- Block-level selection keyed at gate time: needs a coverage build or a re-implementation of cmd/cover's block splitting; function level needs neither.
- Raise testmap's staleness for dependency drift in this task: the parent put the staleness policy with note `testmap-upkeep`.
- Check map freshness against the tier's base: maps are built at main's HEAD,
  so a base that predates a landing to a widely imported package, or a base on a branch commit, misses for every such package.
- Hash the working tree through git's clean filter: needs a git call beside the fabric's readers (PATTERN-fabric-boundary);
  a checkout converting line endings misses instead, a D5 bound.
- Select on a method's receiver or signature change through D4: a method-set change can alter an interface assertion in unchanged code,
  so it is a D5 miss.

## Review rounds

### Round 1

- F1 (BLOCKING): D4 and D5 compare from the map's own `Commit`; the tier-base hash check and the hit-rate risk are gone.
- F2 to F5, F10, F11: D5 compares the importing package's own `Files`; D3 states the spawn scan's bound;
  testmap's dirty and changed-file checks cover every covered package; the marker holds the build's `Commit` and `BuiltAt`;
  an added or removed import is a miss, but an added standard-library or covered-package import;
  same-named `init` functions change together.
- Fixer self-check: D4's rationale names the non-body bound, with an Open risk for non-body landings on main.
- F6, F15, F16: constraint, Notes hint and test-isolation bullet corrected.
- F7: the `gate.yaml` question became a Scope Out line (Q8). F8: the `testmap-upkeep` question reworded.
- F9: the tier cost split is a measured Constraint. F12 to F14: D3, D5 and D7 criteria and labels aligned.

### Round 2

- F1, F2: D2 records every linked package, covered or not, with every file impactset maps to it,
  `go:embed` targets in subdirectories included; D4 and D5 compare that set.
- F3: the non-body change rate on main is a measured Constraint;
  D5 exempts an added const outside an `iota` group, a var with no initializer and a type with no methods;
  the dependency on testmap's rebuild cadence is a question for the parent in Open risks (Q11).
- Fixer self-check: the `iota` group rule, a lower-bound count, the D3 and D4 rationale and criteria aligned, and ignored files kept out of D5.
- F5: D4 compares whole function declarations; a method's receiver or signature change is a D5 method-set miss.
- F6 to F8: a `//go:` directive line is never a comment; D3's union keeps only the tier's tests and drops watch-skipped ones;
  an unparsable changed file is a miss.
- F9: D5's bound names line-ending conversion. F10: testmap repeats its dirty check after each package's tests.
  F11: the gate-slot constraint corrected.
- F4 (decision): with the parent; answered and applied in round 3.

### Round 3

- F1: the parent's rulings applied: D2's staleness rule and its rationale cut, D5's Bound states how long a dependency-drift miss lasts,
  and Open risks state the cadence dependency and order this record's build tasks after `testmap-upkeep`'s design.
- F2: D3's union holds only tests of test files the gate's host compiles; D6 puts the selected `-run` in the run's own args.
- F3: D2's dirty checks are fresh status reads, with a Bound for an edit reverted between them.
- F4: `inert_paths` never exempts a `go:embed` target of a linked package.
- F5: an added top-level declaration named as a predeclared identifier is a D5 miss.

## Question ledger

- Q1 (auto-pick): new scope value or changed `diff`? New value `coverage` (D1); keeps `diff` and avoids a reinterpretation.
- Q2 (auto-pick): which keys default to `coverage`? `darn-write`, `webster-fix`, `code-burler-fix`; `pr-review` keeps `diff` as the last gate before landing.
- Q3 (auto-pick): how to see importer coverage? Cross-package coverpkg in testmap (D2).
- Q4 (auto-pick): tier 2? Import scope as today, as a D5 miss (unmapped tier).
- Q5 (auto-pick): freshness rule? Selection diffs every linked package from the map's own `Commit` (D4);
  a map whose `Commit` is unreadable, or whose importing package's own files differ, is a miss (D5).
- Q6 (auto-pick): non-body changes and new declarations? Non-body change other than comments, formatting and a reordered import block widens,
  as does an added or removed import other than an added standard-library or linked-package one, and any `//go:` directive change;
  a method-set change or an added `init` widens;
  a new plain function, a const outside an `iota` group, a var with no initializer or a type with no methods does not,
  unless its name is a predeclared identifier.
- Q7 (auto-pick): redundancy judgment with cross-package maps? Unchanged, own-package blocks only.
- Q8 (auto-pick): the hub's own `gate.yaml` switch?
  The operator switches the three keys to `coverage` with `lym config gate` from the prime after landing; the run edits no hub config.
- Q9 (auto-pick): selection granularity? Function level from `go/parser` (D4), after the lens review found no gate-time source of block boundaries.
- Q10 (resolved by the parent): may D2 raise testmap's cost ahead of note `testmap-upkeep`?
  No: D2 leaves testmap's staleness rule as it is, and dependency drift goes to that note.
- Q11 (resolved by the parent): land before note `testmap-upkeep` sets a rebuild cadence, or wait for it?
  Land now; Open risks orders this record's build tasks after the task that builds that note's design.
