---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/shedadapters/parentseat_test.go:176-218
    summary: No test row pins the round seed at full nanosecond precision, which D2 requires
    fix: In TestBurlerReviewProducer_Call_ParentAskingRule add a row whose prior ask carries a seed one nanosecond away from the current focus file's modification time, set with os.Chtimes to a time with nonzero nanoseconds, and expect the round to ask again. Add a roundSeed case that sets such a time and expects the RFC 3339 nano string.
  - id: F2
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/shedadapters/parentseat.go:59-73, internal/shedadapters/parentseat.go:109-122, internal/burlerengine/parentseat.go:79-92
    summary: Failure branches of the asking rule and the coverage gate have no test row
    fix: Add rows to TestBurlerReviewProducer_Call_ParentAskingRule for a RecordWrittenAt error (the round is judged as not rewritten), a missing previous report (the round asks again), a missing round-1 focus file and a corrupt asks.json (each asks nothing and posts nothing). Add rows to TestParentCoverageGateEntry_Gate for a done ask whose report file is missing (gate error) and a missing review file (failing result with findings).
  - id: F3
    severity: LOW
    class: consistency
    origin: handler
    location: internal/burlerengine/parentseat_test.go:60-64
    summary: TestParentRulesBlock asserts the static prose of the parent consolidation stencil
    fix: Drop the loop asserting the strings origin parent and lym loom parent-review await in the composed step. Keep the blank-line prefix, no-block and plain-step equality assertions only where TestEngine_Review_TableShape and TestComposePrompt_ClusterRules do not already cover them, and fold any that remain into TestEngine_Review_TableShape.
  - id: F4
    severity: LOW
    class: consistency
    origin: handler
    location: internal/burlerengine/parentreview_test.go:565-572
    summary: TestParentCoverageGateEntry_Shape duplicates the name and PassOnCap assertions of TestEngine_Review_TableShape
    fix: Delete TestParentCoverageGateEntry_Shape and add the Attempts above zero and non-nil Gate assertions to the parent-review row of TestEngine_Review_TableShape at review_test.go:187.
  - id: F5
    severity: LOW
    class: consistency
    origin: handler
    location: internal/burlerengine/parentreview_test.go:543
    summary: TestParentCopyPath pins a file name literal that TestSettleParentAsk reaches only through ParentCopyPath itself
    fix: Mark TestParentCopyPath with a testtiming keep comment directly above its func line, with the reason that it pins the copy's file name suffix .parent.yaml, which the settle test derives from the function under test.
  - id: F6
    severity: NIT
    class: consistency
    origin: lens:code-correctness
    location: internal/loomcli/parentseatwiring.go:1-2
    summary: The file header says every closure reads on a call, but awaitCap is resolved at wire time
    fix: Reword the second header line to say the closures read the key, the history and the routing on a call, and that the chair's await cap is resolved when the seat is built.
---

### [MEDIUM] Seed precision is not pinned by any test

**Location:** internal/shedadapters/parentseat_test.go:176-218
**Issue:** D2 stores the round seed at the file system's full precision and compares it exactly. The fixture seed is a whole-second time and the other-seed rows use a different second, so truncating the seed to seconds would break no test. The focus directive asks whether the seed is compared at nanosecond precision; the code does (`roundSeed` formats with `RFC3339Nano`), but nothing would fail if it did not.

### [LOW] Failure branches without a test row

**Location:** internal/shedadapters/parentseat.go:59-73, internal/shedadapters/parentseat.go:109-122, internal/burlerengine/parentseat.go:79-92
**Issue:** The unreadable-history, unreadable-previous-report, unreadable-seed and unreadable-record branches of the asking rule, and the unreadable-report and missing-review branches of the coverage gate, are behaviours the diff adds that no row exercises. The code on each is correct as traced.

### [LOW] Static stencil prose asserted in TestParentRulesBlock

**Location:** internal/burlerengine/parentseat_test.go:60-64
**Issue:** The loop asserts that the composed step contains the strings origin parent and lym loom parent-review await, which is the static prose of the stencil. The redundancy report lists the test as contained, covered by TestEngine_Review_TableShape and TestComposePrompt_ClusterRules.

### [LOW] TestParentCoverageGateEntry_Shape duplicates a covering test

**Location:** internal/burlerengine/parentreview_test.go:565-572
**Issue:** TestEngine_Review_TableShape asserts at review_test.go:187 that gate[1] is the parent coverage entry and does not pass on its cap, which is what the shape test asserts for the name and PassOnCap. Only the attempts and non-nil gate assertions are not covered there.

### [LOW] TestParentCopyPath needs its keep marker

**Location:** internal/burlerengine/parentreview_test.go:543
**Issue:** The redundancy report lists it as contained in TestSettleParentAsk, but that test builds its expected path with ParentCopyPath itself (parentreview_test.go:481), so the file name suffix is pinned only here. It pins what coverage cannot see and is accepted with a keep marker.

### [NIT] Header overstates when the wiring reads

**Location:** internal/loomcli/parentseatwiring.go:1-2
**Issue:** The header says every closure reads on a call, never at wire time. `awaitCap` is resolved by `chairAwaitCap` when the seat is built.

## Rejected

- lens:code-correctness LOW, a pending-post ask re-posted after its deadline: D3 says a restarted row that finds pending-post posts again to the stored strand and accepts a second mail; the chair's await ends at the first deadline and the settle step closes the ask, so a stale mail does no harm.
- lens:code-tests finding 4, postParentAsk failure paths: the lens could not confirm the gap itself and asked for nothing to be done unless an assertion was missing; the post-error row of the does-not-go-out test asserts the no-parent state.
- lens:code-tests finding 6, no test of the ParentStrand wrapper: it forwards one call to the routing that the NotifyParent tests already cover.
- lens:code-tests finding 7, the non-orch strand claim for PostAsProgram: the program claim is built by creelcli.NewScopedProgramClient from the program name alone, so ignoring the process's strand holds by construction in code outside this diff.
- Declared PATTERN changes, Pattern-Departure trailers, config-layer and refusal rules: the directive declares no PATTERN change, the diff touches neither PATTERN.md nor pattern/, and no commit carries a trailer.
