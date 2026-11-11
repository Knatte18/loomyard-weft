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

### Turn 3 — lens review

Every fork returned a report.

- bound-breaker 1 (BLOCKING): testmap does not drop spawning tests. Fixed: D2 records the spawn verdict, D3 always runs such tests.
- bound-breaker 2 and walkthrough 1 (BLOCKING, merged): no source of block boundaries without a build. Fixed: D4 matches at function level from `go/parser`, with the block keys' enclosing function.
- existing-paths 1 (BLOCKING): dependency hashes in `Files` would make testmap rebuild every map. Fixed: D2 puts them in a field of their own; `stale` and `mapDrift` keep own-package `Files`.
- interactions 2 and existing-paths 2 (merged): dependency drift stays a miss until the age limit. Fixed: D2 adds covered-package drift to testmap's staleness; the cost is a question to the parent in Open risks.
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
- board-scope 1 (decision): D2 ahead of `testmap-upkeep`. Kept as a question for the parent in Open risks.
- board-scope 2 (decision): this hub's `gate.yaml`. Kept as a question for the parent in Open risks.
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
- Leave testmap's staleness at own-package files: a dependency-drift miss would last until the age limit.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): new scope value or changed `diff`? New value `coverage` (D1); keeps `diff` and avoids a reinterpretation.
- Q2 (auto-pick): which keys default to `coverage`? `darn-write`, `webster-fix`, `code-burler-fix`; `pr-review` keeps `diff` as the last gate before landing.
- Q3 (auto-pick): how to see importer coverage? Cross-package coverpkg in testmap (D2).
- Q4 (auto-pick): tier 2? Import scope as today, as a D5 miss (unmapped tier).
- Q5 (auto-pick): freshness rule? Every recorded file hash must match the tier's base (D5).
- Q6 (auto-pick): non-body changes and new functions? Non-body change other than comments, formatting and named imports widens; a new method or `init` widens; a new plain function does not.
- Q7 (auto-pick): redundancy judgment with cross-package maps? Unchanged, own-package blocks only.
- Q8 (open, for the parent): the hub's own `gate.yaml` switch? The record leaves it out; the run edits no hub config.
- Q9 (auto-pick): selection granularity? Function level from `go/parser` (D4), after the lens review found no gate-time source of block boundaries.
- Q10 (open, for the parent): may D2 raise testmap's cost ahead of note `testmap-upkeep`? Recorded in Open risks.
