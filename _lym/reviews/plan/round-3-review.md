---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lym/plan/01-gate-test-map-mode.md:106
    summary: Card 1 deletes the only reader of testMapRun.inert but leaves the field and its assignment in prepare
    fix: >-
      In card 1's Intent paragraph that lists what `testMapRun` drops (the line opening "`testMapRun` drops `selector`, `dirty`, `plan`"),
      add the `inert` field to the dropped fields,
      and add to the `testMapRun.prepare` paragraph that it no longer assigns `r.inert = cfg.Inert`.
      No target changes, since `internal/gatecli#testMapRun` and `internal/gatecli#testMapRun.prepare` are already Edit targets of the card.
  - id: F2
    severity: LOW
    class: design
    location: _lym/plan/03-map-run-tier-bounds.md:58
    summary: mapSideBySide falls back to NumCPU for a tier Parallel below 1, an input the gate config refuses, and a test row pins it
    fix: >-
      In card 3, change the `mapSideBySide` bullet to use the tier's `Parallel` as it stands, dropping "below 1 means `runtime.NumCPU()`",
      and drop the row "a `Parallel` of 0 uses `runtime.NumCPU()`" from the `TestMapSideBySide` bullet.
      No target changes, so no check of the plan gate is affected.
---

### [LOW] Card 1 leaves a write-only `testMapRun.inert` field

**Location:** `_lym/plan/01-gate-test-map-mode.md:106`, the `testMapRun` field drop list; `internal/gatecli/testmap.go:363` and `internal/gatecli/testmapplan.go:148`.

**Issue:** `testMapRun.inert` is assigned once, in `prepare` (`r.inert = cfg.Inert`), and read once, by `testMapRun.headSelector` (`Inert: r.inert` in the `SelectorConfig`).
Card 1 deletes `headSelector`, so the field is written and never read.
The card's list of dropped fields names `selector`, `dirty`, `plan`, `selfCheckMismatches` and `auditMismatches` and omits `inert`.
`gateslot.Config.Inert` keeps other callers (`dotnetscope.go`), so no test-only-code guard fires on it, and the leftover would pass the gate.
The code-quality rule is that something unused is deleted, and the task's scope is to remove every piece only the retired planning reaches.

### [LOW] `mapSideBySide` handles a `Parallel` below 1, which a declared tier never has

**Location:** `_lym/plan/03-map-run-tier-bounds.md:58` and the `TestMapSideBySide` bullet at `_lym/plan/03-map-run-tier-bounds.md:94`.

**Issue:** Card 3 has `mapSideBySide` read the tier's `Parallel` and treat a value below 1 as `runtime.NumCPU()`, with a `TestMapSideBySide` row for a `Parallel` of 0.
The fallback comes from `takeSlot`, which reads `limits.Tier0Parallel`; that value is 0 when the hub declares no tier0.
`mapSideBySide` takes a tier from `mappedTierConfigs`, so the tier is always declared, and a declared tier's `Parallel` is at least 1.
`internal/gateslot/tiers.go` defaults it to 1 and refuses a `parallel` below 1 (line 236), and `applyTierParallel` takes a `tier_parallel` entry of at least 1 only; the package doc at `internal/gateslot/doc.go:152` and `:161` says the same.
The branch and its test row cover an input the config rules out, which the code-quality and testing rules both bar.
A hub with tier1 and no tier0 gets tier1's own `Parallel`, not 0, so the old reason for the fallback does not carry over to the per-tier count.
Round 1 rejected a test-economy finding on this table because its `Parallel` 0 row pins a case no integration subtest reaches; no prior fixer disputed a finding here, and that rejection did not ask whether the input can occur.

## Rejected

- Focus question 1, test-only reachability after cards 1 and 2: no uncovered orphan.
  Callers listed from the code: `testMapRun.audit`, the only caller of `Map.FailedAt` beside `planLockedPackage`, and of `Pool.TryAcquire`, goes whole with `testmapaudit.go`, and `TryAcquire` is deleted by card 1.
  `planLockedPackage` and `headSelector` are the only callers of `PlanUpkeep`, `NewSelector` and `Toolchain.GoRoot`, and `headSelector` calls the selector only; all of them sit in the card's allowlist sentence.
  The `blockTable.profiled` branch of `newBlockTable` is the only caller of `FuncIndex.ProfiledFuncs`, which card 1 deletes.
  `MapTest.Measured`, `readToolchain`, `stampEnvironment`, `readPackageTree`, `moduleDirOf`, `fileHashes`, `readHead`, `ReadRunRecords`, `RunLogDir`, `lock.TryAcquireWriteLock`, `lock.TrackedFileLockPath`, `prebuildLym`, `NestedPackageDirs` and `StampModuleFiles` each keep a production caller outside the deleted code (`redundancy.go`, `environmentStamp`, `linkedTrees`, `report.go`, other lock users, the tier run, `redundantmutants.go`, `readPackageTree`).
  Every other function of `testselect.go`, `upkeep.go` and `declcompare.go` is reachable only from those entry points, so the allowlist sentence's "helpers only they reach" is complete, and the guard names each one when it runs.
  Card 2's deletions leave no helper in a kept file without a caller, and no test outside the deleted test files uses their fixtures.
- Focus question 2, scope base of a computed `--map` run: carried.
  Card 1 calls `computeScope` with a nil green record, and `diffBaseGroups` reads the green record only when it is non-nil, so a computed scope takes the merge base with the parent branch and falls back to whole with no parent branch, as the decision record says.
  `keptTiers` of a request that names no tier is tier0 alone, which matches the card's claim about the shares.
  Card 1's `AComputedScopeTakesNoBaseFromTheGreenRecord` reaches a computed scope with a whole-green record for tier0 present: a green stand-in would give base HEAD and an empty set, where the subtest wants every package built.
  The card 3 subtest `AMapRunWritesNoRunRecordAndAGreenRecordDoesNotStandInForIt` names an explicit target, so it reaches no computed scope, and the card 1 subtest is the one that does.
- `--tags` added to card 1's `--map` refusal list: not scope creep, since the existing `--tags` refusal sends the caller to `--tier`, which `--map` refuses, so the plan's wording avoids a dead end and D1's way forward still holds.
- `TestMapSideBySide` and `TestMapSlotAsk` beside the integration subtests: not a per-symbol test, since their rows (twice the cap, the capped count winning, tier0 one and tier1 two, tier0 asking every slot beside tier1 three) are behaviors no integration subtest covers.
