---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    origin: lens:code-scope
    location: _lym/landing/summary.md:26
    summary: The Removed table gives internal/shuttleengine 6 net lines removed, but the range removes 8 lines and adds 3 in startup_test.go, a net of 5.
    fix: In _lym/landing/summary.md, change the net-lines-removed cell of the internal/shuttleengine row in the Removed table from 6 to 5.
---

### [LOW] Report overstates the shuttleengine removal by one line

**Location:** `_lym/landing/summary.md:26`
**Issue:** The board entry requires the report to list the lines removed per package.
`git diff --numstat 9c5bd131..HEAD` shows `3 8 internal/shuttleengine/startup_test.go`, a net removal of 5.
The table says 6, probably because it was counted before the later doc-comment commits.
Every other package's figure matches the numstat.
The code-scope and code-affected-set lenses and the assistant all raised this same finding; it is merged here.

## Holistic review

- **Scope against the goal.** `runfsm.Order` was production code that only its own subtest reached, and it is not a test helper, so goal point 2 licenses removing it with its subtest and its allowlist entry.
  Every other removal is a test-file helper that nothing reaches, even with `-test`.
  That is the "unreachable function" in the first sentence of point 2.
  The test-helper exception covers helpers that tests still reach, so it does not apply to these.
- **Caller check (focus items 1–3).** A grep over the whole tree finds no reference to any removed symbol in any file, whatever its build tag.
  The only matches are same-named symbols in other packages: hubgeom's `reconcilePairFor`, websterengine's `testGateTiers` constant, `loomcli.resumeFromRunState`, and the `stoppedGUIDs` fields of the sheddriver and shedarbiter fakes.
  None of the changed files carries a build tag.
  No `go:linkname` exists in the code, and the removed symbols are unexported test helpers, so neither reflection nor a registry can reach them.
- **Removal cascade.** Each helper the removed functions called still has another caller: `turnEnds`, `waitFor`, `encodeLivePanes`, `answerFormat`, `paneGenerationFormat`, `commitOffChainOn`, `hubRelative`, the `fakeHandle` channel fields, and `rubricBody`.
- **Coverage (focus item 4).** `TestDerive`'s "X over Y" rows pin every precedence pair that can hold at the same time.
  Cancelled/Cancelling and Starting/Queued are mutually exclusive by their conditions.
  The removed subtest asserted only on the deleted `Order`, so its removal loosens no surviving check.
- **Doc comments (focus item 5).** The rewrites of `Derive`, `fakeHandle.Stop` and `skillStartFixtureWith` stand alone, describe behaviour that did not change, and follow the repository's doc-comment conventions.
- **Commit messages and summary (focus item 6).** The commit messages' claims hold.
  The summary's Kept table names each kept hit and the reason it stays.
  Apart from F1, its per-package figures match the diff.
  The Bounds section states that the Windows and darwin builds were not analysed.
  That limits what the analysis can find, not what the diff removes, and the goal does not require a per-platform run.

## Rejected

- No lens raised a finding I rejected.
  The code-correctness and code-tests lenses reported none.
- Considered and dropped, as a loosened check: the removal of `OrderListsEveryStateHighestFirst`.
  Goal point 2 asks for the tests covering only a deleted function to go with it, and `TestDerive` still pins the precedence.
- Considered and dropped, as a scope gap: the Linux-only analysis bound.
  Every removed symbol has no reference in any platform's files, and the goal names no per-platform run.
