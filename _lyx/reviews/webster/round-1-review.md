---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: internal/failuretrace/trace.go (NewTracer)
    summary: NewTracer has no test of its own; its unreadable-plan and missing-state error paths are unpinned
    fix: Add a NewTracer test in internal/failuretrace/trace_test.go with one row for a missing plan directory and one for a missing Webster state, each asserting the returned error, so the tests need no quarry.
    origin: lens:code-tests
  - id: F2
    severity: NIT
    class: consistency
    location: internal/failuretrace/doc.go:5
    summary: package doc says failuretrace is told absolute paths and derives none, but NewTracer derives the plan, Webster and go.mod paths from anchorPath and worktreeRoot
    fix: In internal/failuretrace/doc.go reword the sentence to say the tracer is told the worktree root and anchor path and takes every other path from the owning packages' own path functions.
    origin: lens:code-pattern-fit
---

### [LOW] NewTracer has no test of its own

**Location:** internal/failuretrace/trace.go (`NewTracer`)
**Issue:** `trace_test.go` holds `TestTrace` over `Trace` and `locations_test.go` holds `TestLocations`.
The landing seam tests stub `TraceFailure`, so nothing exercises `NewTracer`'s own behavior: an unreadable plan and a missing Webster state both return an error.
Card 26 states a missing plan or state is an error.

### [NIT] failuretrace doc claims it derives no paths

**Location:** internal/failuretrace/doc.go:5
**Issue:** The doc says the package "is told absolute paths and derives none".
`NewTracer` joins `go.mod` onto the worktree root and derives the plan, Webster and scratch directories from `anchorPath` through `planparser.PlanDir`, `websterengine.Dir` and `websterengine.ScratchDir`.
`PATTERN-told-geometry` binds engines and this package is not one, so this is a wording mismatch only.

## Rejected

- code-correctness: `AppendAmendment` writes with a plain `os.WriteFile`, so a crash mid-write leaves an undecodable record. The decode error names the fix and this is internal code, so no atomic write is warranted.
- code-correctness: `..` segments pass `tracedLocation` and `place`. No escape is shown, since the path only feeds a revision lookup.
- code-affected-set: `batcher.csharpResignTargets` reads Edit groups only. `extends-resign` refuses an arrow under Extends, so an Extends group never carries one.
- code-tests: the "Webster holds no commit range for this card" line has no row. `publishfailure_test.go:406` and `:415` assert it.
- code-tests: `PatternHeadroomIsReportedThroughTheReworkGate` may duplicate the gate table. It uses a different gate constructor and no covering test was named, so it is not a redundancy finding.

Settled without a finding:

- Cards 25 to 27: Publish failure note names traced cards, later ranges and untraced locations, via `EncloseAt` and `planparser.TargetHolds`.
- Cards 11 to 16: one writer, `decision add --amends-plan`, with the reader and removal seams matching their cards.
- Cards 6 to 9: one `planparser` pair rule, used by Webster's batch-order check and `tools/plandag`.
- Cards 2 to 5: `PATTERN.md` is unchanged and the `EncloseAt` bullet lands in the glyph-conversion background file.
- Cards 17 to 21: the stencil changes were not read line by line, so no check of dropped or weakened rules was made.
- No `Pattern-Departure:` trailer is on any commit in the range.
