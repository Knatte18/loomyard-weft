---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/finalize.go:754
    summary: Accepted is a bare bool not bound to a HEAD, so a fixer change made after the operator accepted lands with no cap check and no fix review
    fix: "In internal/landingshed/fixerattempts.go replace the Accepted bool with an AcceptedHead string, the task HEAD the operator accepted. In AcceptFixReview (fixaccept.go) record that HEAD, taken after the description commit moves it, or have the circling accept wiring write it once the commit is made, or compare the Baseline..HEAD range with the description commit excluded. In judgeFixerChange (finalize.go) proceed only when rec.AcceptedHead equals the current head, as Reviewed is compared, so a later fixer turn or an operator commit goes to the fix review. gateAndFix keeps clearing nothing on an accepted resume. Add a finalize_gate_test.go row seeding an accepted record, a failing gate then a passing one with a fixer change over the caps, and assert the fix review runs."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/landingshed/finalize.go:629
    summary: The resume path with a pending fix review report has no test row, neither the gateAndFix branch that runs the fixer on a passing gate nor runFixer clearing FixReview after a fixed end
    fix: "In internal/landingshed/finalize_gate_test.go add a seeded-resume row with FixReview set to the review path, Spent below the cap and a passing gate, expecting one fixer session whose fix_review_path value is that path. Add a row where the fixer ends fixed and the review session is made to fail to start, asserting the attempt record on disk holds an empty FixReview so a resume does not rerun the fixer."
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/specs/loom-plan-spec.md:329
    summary: The spec still says the landing rows halt only when the fixer cap is spent or its change is over the caps, which Finalize no longer does
    fix: "In contracts/specs/loom-plan-spec.md line 329 reword the clause to say the rows halt only when the fixer cap is spent, a Parent-Align fixer change is over the caps, or Finalize's fix reviews run out of rounds."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/landingshed/fixaccept_test.go:33
    summary: AcceptFixReview's retry branch, its missing-description error and the findings a no-verdict halt records are not pinned by any row
    fix: "In internal/landingshed/fixaccept_test.go add a row seeding the description with the heading already present and Accepted false, expecting no second append and Accepted true afterwards, and a row with a missing description file expecting an error and Accepted false. In finalize_gate_test.go on one no-verdict row of the fix review end table assert rec.Findings holds the verdictSummary line."
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/loomrecipe/tapestryrecipe_test.go
    summary: Darn and sampler recipe tests assert Finalize declares no on_reroute but the tapestry recipe, which also lost the edge, has no such assertion
    fix: "Add an assertion to the tapestry recipe test in internal/loomrecipe that the Finalize row's OnReroute is empty, in the style of darnrecipe_test.go:190 and samplerrecipe_test.go:74."
  - id: F6
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/battenshed/innerrun_halted_test.go:614
    summary: Tests in battenshed, shedverbs and sheddriver still use a Finalize reroute-cap hand-back as their example although HandsBack no longer produces one
    fix: "Rename the Finalize reroute-cap rows and fake predicates in internal/battenshed/innerrun_halted_test.go, internal/shedverbs/status_test.go and internal/sheddriver/drive_test.go to a row that can still reroute, such as Webster, or drop the Finalize-specific row where Webster is already covered."
  - id: F7
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:581
    summary: Two overview passages describe lym loom circling accept as settling only a review segment escalation
    fix: "In docs/overview.md lines 581 and 690 add a clause that accept also settles a run halted at Finalize after its fix reviews ran out of rounds."
  - id: F8
    severity: NIT
    class: design
    origin: lens:code-tests
    location: internal/landingshed/finalize_gate_test.go:630
    summary: The resume half of the spent-cap test repeats the seeded-resume row for a failing gate with attempts spent
    fix: "In internal/landingshed/finalize_gate_test.go drop the second call of the test named a spent cap halts naming the commits, keeping its first-call halt and Halt assertions, since the seeded-resume row a failing gate halts as a spent cap already pins a resume over a failing gate with no fixer spawn."
---

### [MEDIUM] Acceptance is not bound to the accepted HEAD

**Location:** `internal/landingshed/finalize.go:754`, `internal/landingshed/fixerattempts.go:36`, `internal/landingshed/fixaccept.go:381`.
**Issue:** `judgeFixerChange` proceeds when `rec.Accepted` is true whatever the HEAD is.
The flag survives until the record is removed at landing, and `gateAndFix` keeps it on a plain resume.
A resume re-merges the parent and runs the gate; a gate failure then spawns the fixer again, because `Spent` is below the cap, and the new change skips the cap check and the fix review.
Operator commits made between the accept and the resume pass the same way.
The board entry asks that an over-cap fixer change always reach the fix review and that an acceptance land the change as it stands with its findings written down; this lets an unreviewed change land with no findings recorded.
The accepted-then-changed-tree case in the focus list is this one.

### [MEDIUM] The pending-findings resume path is untested

**Location:** `internal/landingshed/finalize.go:629`, `internal/landingshed/finalize.go:705`.
**Issue:** `gateAndFix` runs the fixer on a passing gate when `rec.FixReview` is set and attempts remain, and `runFixer` clears and persists `FixReview` after a fixed end.
No `finalize_gate_test.go` row seeds `FixReview`; `fixerattempts_test.go` only round-trips it.
Breaking either piece would make a resume rerun the fixer on a fixed tree, or skip the fixer a pending rejection needs, with every test still green.

### [LOW] Spec still describes the old halt rule

**Location:** `contracts/specs/loom-plan-spec.md:329`.
**Issue:** The line says the landing rows halt only when the fixer cap is spent or its change is over the caps.
Finalize now halts on an over-cap change only after its fix reviews run out of rounds.

### [LOW] Accept path coverage gaps

**Location:** `internal/landingshed/fixaccept_test.go:33`.
**Issue:** `TestAcceptFixReview` covers the already-accepted early return but not the branch that skips the append because the heading is present while `Accepted` is false, which the doc comment promises as retry safety.
A missing description file has no row.
The findings a no-verdict halt writes through `openFindings` are asserted on no row, though `AcceptFixReview` copies them into the description.

### [LOW] Tapestry has no no-reroute assertion

**Location:** `internal/loomrecipe/tapestryrecipe_test.go`.
**Issue:** The darn and sampler recipe tests assert Finalize's empty `OnReroute`; tapestry's recipe lost the same edge and nothing pins it.

### [NIT] Tests still model a Finalize reroute-cap hand-back

**Location:** `internal/battenshed/innerrun_halted_test.go:614`, `internal/shedverbs/status_test.go:483`, `internal/sheddriver/drive_test.go:154`.
**Issue:** They use injected predicates, so they pass, but they name a halt `HandsBack` no longer produces.

### [NIT] Overview describes circling accept for Bouncer rows only

**Location:** `docs/overview.md:581`.
**Issue:** Lines 581 and 690 say `accept` settles a review segment escalation; the verb now also settles a Finalize halt at the fix review round cap, which line 597 already describes.

### [NIT] Duplicate resume coverage in the spent-cap test

**Location:** `internal/landingshed/finalize_gate_test.go:630`.
**Issue:** The test's second call, a resume over a failing gate with attempts spent and no fixer session, is what the seeded-resume row "a failing gate halts as a spent cap" already asserts.

## Rejected

- Appending to the change description is a PATTERN-sole-parsers violation (handler): `AcceptFixReview` neither parses the summary's title or body nor declares its filename, which the caller takes from `summaryparser.Path`; it checks for its own section heading and appends, and the correctness lens confirmed `summaryparser` still parses the result.
- A failed description commit leaves a dirty `summary.md` and the resume's clean-tree halt does not name re-running accept (lens:code-correctness F2): the refusal the failed verb prints already ends with `circlingWayRetry`, which says to re-run the verb, so the operator is told at the point of failure.
- `README.old.md` still says a conflict reroutes to Parent-Align (lens:code-affected-set): an archive copy outside the diff's subject.
- Stencil wording: the fixer stencil only adds the findings instruction when a report is named and the orch stencil drops only wording that rested on the reroute; neither drops a rule the entry did not ask to change (lens:code-affected-set, handler).
