# Round 1 fixer report — discussion

Every finding was checked against `internal/planglyph/callercoverage.go`, `internal/websterengine/cardgate.go`, `internal/planparser/targetdirs.go` and the rubric, and each premise held.

## Fixed

- **F1 (BLOCKING):** D5 is rewritten: the walk keeps today's file set and scans nested modules, whose files the root load leaves untyped, so they go through D3.
  D5 now states its bound: method callers there are informational, and package-level callers stay blocking.
  D2's bound is true again.
  Scope's planglyph bullet and the nested-module acceptance criterion now say the files are scanned through the fallback.
  In the support log, Q6 is revised and skipping nested modules is a rejected alternative.
- **F2 (BLOCKING):** D9 now binds the proposed fix, never the finding.
  When the reviewer sees no admissible fix, it still reports the defect and names the constraining check, and a `Bound:` states that no defect becomes unreportable.
  The rubric acceptance criterion and Q10 match, and dropping the finding is a rejected alternative.
- **F3 (LOW):** D1 gains a `Bound:`: interface-dispatched calls are no longer reported.
  The card gate's root build catches the assignment-site break, and a nested module's break is caught only by a command run inside it.
- **F4 (LOW):** D6 counts a Rename pair moving a `go.mod` as a `Create` of the new side and a `Delete` of the old side, and its rationale now covers it.
  The card-gate acceptance criterion and Q9 name the rename case.
- **F5 (LOW):** the deduplication requirement moved into D1 and the flag-value rule into D8, and both bullets left Open risks.
  The `verify-nested-module` acceptance criterion now includes the `-o ./bin` case.
- **F6 (NIT):** D8 states that `go -C <dir>` resolves against the directory a preceding `cd` set, and against the root otherwise.

## Disputed

None.

## Deferred

None.
