```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: attack every bound, guarantee and existing-code claim in the decision record and support log.

**Claims attacked**

- D1 bound ("skips tests of importing packages only; `finalize`, scope `all`, runs every test before landing"): held. The hub's `gate.yaml` has `finalize` at tiers 0–2, scope `all`.
- D1 "never adds a package `diff` would not run": held.
- D1 "explicit targets are never narrowed": held. `withGateScope` returns early on targets (scope.go:108).
- D2 "`BuildBinary` already takes a comma-separated coverpkg list": held (runner.go:62-64).
- D2 "redundancy judges against kept tests' own-package blocks only", with the Notes hint "reuse `testcover.PackageBlocks`": broken as a hint (F4).
- D2 "`mapDrift` keeps comparing own-package files only": held. It takes `m.Files` only (redundancy.go:316, 349).
- D3 "a package added only because its tests use the `lym` binary runs whole": held. `impactedSet` adds lymbin packages only when `cmd/lym` itself changed (graph.go:119-121).
- D3 "every test its map marks out-of-process or unresolved" covers out-of-process tests: broken (F2).
- D4 "function names come from the same enclosing-function naming the block keys use": held. `funcKeyName` and `receiverTypeName` strip pointers and type arguments; closures fall in their enclosing FuncDecl; package-level func literals get `(package)`, which D5's outside-body rule widens. Several `init` functions share one name, which only widens.
- D4 "selects a superset of the tests a block-level match would": held.
- D5 "a new plain function is reached only through a changed caller": held. A reference from a package-level initializer is outside a body and widens.
- D5 "comments, formatting and named imports outside bodies are no miss": broken (F3).
- D5 freshness check ("for a covered package, the recorded file set or hash differs at the tier's base"): broken by ambiguity (F1).
- D5 bound ("a test reaching a covered package's function it did not reach at map time, through unchanged code"): held. Package `init` blocks run in every test's profile, so a linked package with init code is "covered" and its drift widens.
- D6 "the anchored `-run` form already exists in `gateslot.FailedTestsCall`": held (tiers.go:440).
- D6 "a `-run` green entry never serves as a base and never stands in for a whole run": held. `newestWholeAncestor` skips entries with args (green.go:205), and `argsMatch` matches non-empty args only exactly (green.go:224-229).
- D6 "the command budget" (Notes: in green.go): held as a pointer. `greenCommandBudget` is 6000 bytes, but it bounds a carried remainder's targets, not a `-run` pattern. That is a reuse, not an existing pattern budget.
- D8 "reads only through `testcover.ReadMap`; a failed read is a miss": held. `ReadMap` reads under a lock and `WriteMap` is atomic (coveragemap.go:61-72).
- Constraint PATTERN-fabric-boundary ("base file content for D4 is read through the path the impactset diff already uses, never a new raw git call"): broken (F0).

**Findings**

- **F0 — MEDIUM, design, Constraints (PATTERN-fabric-boundary bullet).** The constraint names an existing path that reads base file content, and no such path exists. impactset reaches git only through `gitexec.Run` with `rev-parse`, `merge-base`, `diff --name-status -z` and `ls-files` (derive.go:22,27; targets.go:109,118). None of these returns file content. The existing content read is `fabricengine.Fabric.CodeFileAt` and `CodeDirAt` (codereads.go:16,27), which testmap's `fileHashes` uses (testmap.go `fileHashes`). As written, the constraint either cannot be met or pushes Plan toward a new `git show` through `gitexec` in impactset. Fix: name `Fabric.CodeFileAt` as the base-content read.
- **F1 — MEDIUM, design, D5 bullet 2 with D2.** It is unclear whether the importing package's own `Files` is checked against the tier's base. D2 records the hashes "per covered package other than its own" in a new field. D5 checks "for a covered package", so an implementer can check only the new field. Counterexample: on main, after B's map is built, B's test T is edited to call `A.G`. A branch based on that main then edits `A.G`. B is not a changed package, T is still recorded under its name, so D3's "absent from map" does not fire. The map shows T not covering G, so T is not selected. Own `Files` (which includes `_test.go` per `fileHashes`) or `MapTest.FuncHash` would catch it, but no decision names either. Fix: D5 states that the importing package's own `Files` is compared too.
- **F2 — MEDIUM, design, D3 bullet 3 with D1's bound.** The out-of-process guarantee rests on `ScanSpawns`, whose blind spots the record does not state.
  - `analyzeCall` returns an empty verdict for any call through an import name, so a call into an in-module, non-testkit package is not followed (spawnscan.go `case *ast.SelectorExpr`, `isImport` → `return out`).
  - Every test in a tier0 file gets an empty verdict (spawnscan.go:185-187).
  - Its own doc bound excludes hooks and aliases that make git run module code (spawnscan.go:174).
  - Counterexample: a tier1 test of B calls `A.Launch()`, production code that execs the prebuilt `lym` from the tier's prebuilt-lym env. The scan marks nothing, the child's coverage never reaches T's profile, and a change in the code the child runs leaves T unselected.
  - The landing gate still bounds this, but D1/D3 should state the let-through: tests that run module code in a child the scan does not see.
- **F3 — LOW, design, D5 bullet 4 ("named imports … no miss").** Removing a named import can drop a package from the binary, and its `init` with it. Counterexample: A's `F` drops its only use of `plugin`, whose `init` registers a handler. F's body changed, so only tests covering F are selected. A B test that resolved the handler through a registry, without covering F, now loses that registration and is not selected. Fix: a change that removes or adds a named import whose package then leaves or enters the build counts as a miss. Alternatively, D5's bound states the case.
- **F4 — LOW, consistency, Notes for the plan writer (last bullet), with D2.** The hint "the redundancy judgment's own-package filter can reuse `testcover.PackageBlocks`" is wrong. `PackageBlocks` filters `[]ProfileBlock` by `path.Dir(b.File)` (profile.go:78-87). The judgment that D2 changes reads the map's `[]BlockKey` (redundancy.go:592-593, `p.m.Blocks[index].String()`), where own-package keys hold only a file name. The filter needed is "keys without an import path". Fix: drop or correct the hint.

**Tool calls**

1. `internal/impactset/graph.go` `impactedSet`/`addBinaryBuildingTests` and `internal/testcover/runner.go` `BuildBinary`: the manifest had neither the rule that adds lymbin packages nor `BuildBinary`'s body, which the D3 and Notes claims rest on.
