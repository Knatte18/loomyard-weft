---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: internal/planglyph/deleteorder.go:317
    summary: Partitioned-member behavior in buildEditRegions, coverageSubjects, coveredByTarget and the resign-interface-method arm has no test
---

### [MEDIUM] Partitioned-member behavior added to four consumers of `answerSymbols` is uncovered

**Location:** `internal/planglyph/deleteorder.go:317` (`answerSymbols`), with consumers `buildEditRegions` (`deleteorder.go:118`), `coverageSubjects` (`callercoverage.go:214`), `coveredByTarget` (`callercoverage.go:489`) and the `resign-interface-method` arm of `planGatePass` (`plangate.go:86`).

**Issue:** Card 12 makes `answerSymbols` return a partitioned member's candidates as its declarations, and threads the `partitioned` map into five call sites.
The diff adds two tests for it: a `TestLaterDeleteReferences` row, whose Delete target is partitioned and whose later card edits a whole file (`part/c.go#`), and a `TestGlyphChain_RedundantFile` row.
Neither reaches a partitioned member as an Edit member glyph in `buildEditRegions`, as a deleted or re-signed subject in `coverageSubjects` (`caller-uncovered`), as a covering target in `coveredByTarget`, or as an interface method in the `resign-interface-method` arm.
Those are behaviors the diff adds, and no existing test covers them: the pre-existing rows in `deleteorder_test.go` and `glyphchain_test.go` use unpartitioned fixtures, where `partitioned[ref]` is false and the new branch never runs.
A revert of any one of those four `partitioned[...]` arguments to `false` would pass the whole suite (`lyx gate test` of `planglyph`, `shuttleengine`, `websterengine`, `webstercli`, `contracts/stencils` and `cmd/lyx` is green).

**Fix:** Extend the existing tables rather than adding top-level tests, since the `planglyph` budget row is already raised to its limit.
Add a `TestLaterDeleteReferences` row whose later card edits the partitioned member by member glyph and references it inside its span.
Add a caller-coverage row, in the existing caller-coverage table, for a deleted partitioned member referenced from a file that a covering target does and does not span.
Add a `resign-interface-method` row for a partitioned interface method.

## Focus answers

- Cards 10 to 13, every pass that reads an `ambiguous` answer: `statusFindings`, `createFindings`, `doneCheckVerdicts` and `renameDeclSource` call `classifyAmbiguity` directly; `LaterDeleteReferences`, `planGatePass`, `buildEditRegions`, `coverageSubjects` and `coveredByTarget` go through `partitionedAnswers` and `answerSymbols`. A grep of `StatusAmbiguous`, `.Candidates` and `.Symbols` over `internal/planglyph` finds no other reader, so no pass still treats a partitioned member as ambiguous. The Delete direction, the Rename old side and the Create inversion keep blocking by design, as cards 10, 11 and 13 state.
- Card 1: `startedFresh`, `gatedArrival` and `countsAtOnce` have no remaining reader, test or doc; `AwaitedShellPrefixes` is still read by `waitmark.go`. A gated run returns no outcome from `expiredTurnEnd` whatever is outstanding.
- Cards 5 to 7: `recoverSpawn` appends the session id, `BeginBatch` carries the list, `loadRunWrites`, `writtenWorktreePaths` and `ownTrackedPaths` all read `Recoveries`, and a `stuck` or `dead` digest leaves `recover-batch` through `output.ErrFields` with the digest fields.
- Cards 2, 6 and 7: no test or doc still asserts the old text; the remaining "a fork wrote" strings are the fork-kind case.
- Per-card mechanical check: the only Delete groups are card 1 `Run.countsAtOnce` and card 6 `noteForkWroteLast`, and neither name has a remaining reference. The cards carry no Rename group.
- Test economy: `TestClassifyAmbiguity` is one table-driven test, and the card 10, 11 and 13 additions extend existing tables or add a subtest; the one gap is F1.
