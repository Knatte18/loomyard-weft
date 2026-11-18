---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: _lym/landing/summary.md:26
    summary: The shuttleengine row overstates the net removal by one line.
    fix: Change the internal/shuttleengine net-lines-removed cell in _lym/landing/summary.md from 6 to 5.
---

### [LOW] Removal count is stale

**Location:** _lym/landing/summary.md:26
**Issue:** The row reports six net lines removed from `internal/shuttleengine`.
The committed range contains eight deleted lines and three added lines in `startup_test.go`, a net removal of five.
The final comment edits changed the count the required report describes.

## Review evidence

Reviewed the supplied diff pieces and commit messages over `9c5bd13101af1686192cd463afa2b7b583b72697..HEAD`.
The board entry and decision record require deleting unreachable functions and their exclusive tests.
Base-tree and current-tree searches found no surviving callers of the deleted helpers, including platform-specific files.
No reflection lookup or linkname dependency on a removed symbol was found.
The same-named helpers in other packages are separate definitions.

`Order` had only its deleted subtest as a caller.
The state table, derivation body, precedence rows and cancelled-record recovery rows remain unchanged.
The burler stop path retains its channel and error handling; its deleted log had no surviving reader.
The startup fixture retains its implementation and direct callers.
The helper exception protects support that tests use, so removing these uncalled helpers follows the scope.
No surviving assertion was loosened, and no test or timing bound was added.

All four lens reports returned; the affected-set finding above also matches the handler's count check.
No findings were rejected.

## Validation limit

Attempted `timeout 3600s lym gate test --wait --tier 0 ./internal/runfsm` in the foreground.
The gate could not acquire a slot because its holder record under the hub's `_bolt/.lym/gate` was read-only.
No tests ran in this review attempt; this environmental refusal is not a code finding.
