---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: pattern/PATTERN-test-speed.md:247
    summary: the test economy section still describes the module-wide judging and a "covers nothing" list that the package view changed
    fix: In pattern/PATTERN-test-speed.md's "Test economy" section, add one sentence saying `go run ./cmd/testtiming -redundancy` judges candidates over the blocks of the package's own files and adds a "Contained module-wide" column. Widen the "no coverage" bullet "a test that is skipped, failed or covers nothing" to "covers nothing of its own package".
    origin: lens:code-affected-set
  - id: F2
    severity: LOW
    class: consistency
    location: docs/benchmarks/test-redundancy.md:3
    summary: a -pkg splice keeps the old report header, which describes the module-wide judging, above four-column sections
    fix: Make spliceReport in cmd/testtiming refresh the report header from renderReport's header text on a splice run, or regenerate docs/benchmarks/test-redundancy.md in this diff. The header and the sections of one report must describe one judging method.
    origin: lens:code-affected-set
  - id: F3
    severity: LOW
    class: design
    location: internal/testcover/coveragemap_test.go:20
    summary: TestWriteMapReadMap never sets BuildFailure and never shows a second WriteMap replacing the first record whole, both D4 behaviours
    fix: In TestWriteMapReadMap, after the first round trip write a second Map that sets BuildFailure and holds fewer Tests and Blocks, read it back, and assert it equals the second map, with none of the first map's tests left.
    origin: lens:code-tests
  - id: F4
    severity: LOW
    class: design
    location: cmd/testtiming/redundancy_test.go:29
    summary: no judgePackage case pins a candidate that is also contained module-wide, nor a kept test that is a candidate
    fix: In TestJudgePackage, add a run whose package and module blocks are covered by another test, and assert candidate true and moduleContained true. Give a keep reason to a candidate in the keeps map, and assert its verdict keeps candidate true and carries the keep text.
    origin: lens:code-tests
  - id: F5
    severity: LOW
    class: design
    location: internal/testcover/runner_test.go:77
    summary: no TestRunAlone row for a skipped test that wrote a profile, so the profile parse for a skip is unpinned
    fix: Add a stand-in test name to testmain_test.go that prints a SKIP line and writes a profile, and add a TestRunAlone row expecting Action skip with that profile's blocks.
    origin: lens:code-tests
  - id: F6
    severity: LOW
    class: design
    location: internal/testcover/blockkey_test.go:38
    summary: TestKeyBlocks has no case for blocks in two files, whose ordinals count per file, or for a generic receiver
    fix: In TestKeyBlocks, add a second source file with a block of the same text as one in the first, and assert both keys carry ordinal 0 and their own File. Add a method on a generic receiver `T[K]` and assert its key names `T.Method`.
    origin: lens:code-tests
---

### [LOW] Test economy section describes the old judging

**Location:** pattern/PATTERN-test-speed.md:247
**Issue:** The section says a candidate has its covered blocks covered by other tests and that the "no coverage" list holds a test that covers nothing. Under the package view a candidate is judged over its own package's blocks, the report carries a module-wide column, and a test that covers only other packages' blocks is listed as "no blocks". The repository's own rules ask that the docs a change requires land in the same diff.

### [LOW] A splice leaves a stale report header

**Location:** docs/benchmarks/test-redundancy.md:3
**Issue:** The header text is written only by a whole run. The first `-pkg` run after this change puts four-column sections under a header that describes the module-wide judging. The change description admits the old sections stay, and does not name the header.

### [LOW] Map tests miss BuildFailure and whole replacement

**Location:** internal/testcover/coveragemap_test.go:20
**Issue:** D4 asks that a map written and read back yields the same fields and that a write replaces the old record whole. The only written map leaves `BuildFailure` empty, and no second write shows the old tests gone.

### [LOW] judgePackage keeps two copies without a covering case

**Location:** cmd/testtiming/redundancy_test.go:29
**Issue:** `TestJudgePackage` covers a candidate not contained module-wide. The `moduleContained: true` path and a kept candidate, which the decision record says stays listed with its classification unchanged, reach the report only through hand-built verdicts in `TestRenderPackage`.

### [LOW] Skip with a profile is untested

**Location:** internal/testcover/runner_test.go:77
**Issue:** `RunAlone` parses the profile of a test that skipped after writing one. The four rows cover a pass, a fail, a skip without a profile and a pass without a profile, so that branch has no row that would fail if it broke.

### [LOW] Block keys are untested across files and generic receivers

**Location:** internal/testcover/blockkey_test.go:38
**Issue:** The ordinal grouping per file and the `IndexExpr` and `IndexListExpr` receiver cases of `receiverTypeName` have no case, though both are part of the D3 key the map and the gate step compare.

## Rejected

- `Toolchain.Env` nil behaves differently in `RunAlone` and `goOutput` (lens:code-correctness, LOW): the doc says `Env` is the whole environment and the one caller passes a full one; the engine is not a public API, so a caller that leaves it unset is internal misuse.
- No test of `measurePackage` (raised and dropped by lens:code-tests): the decision record's constraint bars a test that launches `go test`, and the pre-change tree had none.
- Missing test for `lymdirs.TestCoverDirName` (lens:code-tests): a constant with no behavior of its own.
- Lock path of the map comes from `lock.TrackedFileLockPath` for a file under the never-tracked `.lym` (own reading): the function mirrors the path under the checkout's `.lym/locks`, which the PATTERN entry asks for, so no violation.
