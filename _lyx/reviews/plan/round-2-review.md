---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: _lyx/plan/05-shuttleengines-registry.md
    summary: modelspec.EngineNames is created as a sorted copy, but its only test compares it with the registry's single name, so neither the copy nor the sort is pinned
    fix: In card 5, extend the planned `TestNames_EqualModelspecEngineNames` instead of adding a test. After the equality assertion, overwrite the first element of the slice `modelspec.EngineNames()` returned, then assert that a fresh `modelspec.EngineNames()` still equals `Names()`. This pins that the caller gets a copy of the closed set. State in the card that sort order cannot be pinned while `claude` is the only name. It adds no top-level test, so the test-budget row stays at 3. It changes no target list and no verify command, so no plan-gate overlay applies.
---

### [LOW] modelspec.EngineNames' copy contract has no test

**Location:** `_lyx/plan/05-shuttleengines-registry.md`, the `modelspec#EngineNames` Create target and the `TestNames_EqualModelspecEngineNames` bullet.

**Issue:** The card's Intent says `EngineNames` returns "a sorted copy of `knownEngines`' keys, so a caller cannot change the closed set" (D12).
The only test planned for the symbol is `TestNames_EqualModelspecEngineNames`, which asserts `shuttleengines.Names()` equals `modelspec.EngineNames()`.
`modelspec` is a different package from the card's own package, the plan adds no test file under it, and its budget row is untouched.
With `claude` as the only name, that equality passes whether or not the function copies, so the copy half of the contract is covered by no test.
Sort order cannot be pinned with one name, so only the copy half is a fixable gap.
The fix keeps the test inside the existing function, so `PATTERN-test-economy` is respected.

## Rejected

- Card 4 (focus 2): every symbol it creates is covered in the same card. `TestNewModelChoice` maps all four fields of `ModelChoice` through `NewModelChoice`, and its rows cover an absent key and an unrelated key. The card bundles no public symbol of another package. Not a defect.
- Cards 7, 11 and 12, unresolved-default guards (focus 1): each states the lookup-error branch as a fail-loud wiring guard with no per-row test. The claim holds, because the plan's registry `build` refuses a `cfg.Engine` that names no registered engine, and every production set comes from `shuttleengines.New` (checked against the production construction sites in the tree). Not a defect.
- Card 11, the engine-without-fork-audit refusal and the registry's forks guard (focus 1): the refusal's reaching input is named, a registered engine without `ForkAuditor`, and so is its test, `TestAuditEngineFrom`. The guard's test row enters through `build` with an injected constructor map, which the registry's `New` cannot produce but `build` is built to take. Not a defect.
- Card 12, the engine-without-capability branch (focus 1): a registered engine without `ResumeContextRenderer` cannot come out of the registry's build today. The card names a test whose receiver holds a hand-built set, and the decision record's acceptance criteria require exactly that test, so the branch has a named input and a named test and is not a fail-loud wiring guard in the refusal spec's sense. Not a defect.
- Card 14, file-level target on `seam_enforcement_test.go` (focus 3): the Intent changes the file header comment, the named test, its doc comment, its failure message and its `//testtiming:keep` reason. The header sits above `package` and no member glyph's span holds it, so only the file's self glyph reaches every changed part. A member glyph beside the file is refused by `redundant-file-target`. `engine.go` and `doc.go` are listed whole for the same reason. Nothing else in the tree states the old shuttleengine-only denylist: the `cliwire` and `orchengine` seam tests keep their own allowlist rules, which D13 leaves alone. Not a defect.
- Card 11, the five websterengine test files listed as `Edit` targets with no change stated in the Intent (`auditledger_test.go`, `archivereset_test.go`, `runwrites_test.go`, `suspect_test.go`, `runlevel_test.go`) look unneeded, since they pass `nil` or a `*shuttlefake.Engine`. The overlay with the five removed reports `caller-uncovered` for each, so the gate requires them. Not a defect.
