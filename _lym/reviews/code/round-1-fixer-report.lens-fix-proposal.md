Findings:

1. **MEDIUM** — `pattern/PATTERN-owner-machine.md`, the F2 edit. The two edits leave siblings contradicting the new rule.
   - Line 29 still reads "A resume that judged its owner without origin leaves the origin-owed marker, which stays until a pass that reached origin exits 0". The new exception says the pair-gone Teardown resume judges the owner without origin and writes no marker.
   - The verdict bullet at line 13 still reads "The verdict that gates a resume … is judged against origin inside `ReconcileOrigin`". The same applies to the header's "resume with origin unreachable reading the local Bolt".
   - Concrete case: a reader applying line 29 expects the origin-owed marker after a pair-gone teardown resume, but `ownerPreflight` writes none. The test asserts `OriginOwed` is false.
   - Fix: qualify line 29 with the same exception. Consider whether the header sentence and line 13 need the same qualifier.

2. **LOW** — `internal/battencli/owner_test.go`, the F1 rows. `TestOwnerPreflight_TeardownWithPairGone` now also holds a `pairPresent: true` row, so the name no longer describes the whole test.

3. **LOW** — `pattern/PATTERN.md`. The entry restates line 3, which still says the resume verdict is judged inside `ReconcileOrigin` or from the local Bolt on unreachable origin. The index file was not found at `pattern/PATTERN.md`, so whether the entry still holds is unchecked.

Checked with no finding: F3, F4 and F1 (the pair-present row fails if the `!present` check is removed).

Tool calls: none.
