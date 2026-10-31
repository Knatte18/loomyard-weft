---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/orchengine/watcher_compact_test.go:501
    summary: No test fails if the requested compact cycle stops recording PointerMarkSince from the previous baseline, because the field then stays zero and the new test's mark still matches
    fix: In TestCompact_HookMarkStampedBeforeTheBoundaryIsNotTypedAgain's file, add a case (a row or a second mark in the same test) where the compact env has a set CompactionBaseline and the mark is older than it; expect the pointer typed. Do this by setting st.CompactionBaseline through the env before reachCompacting and writing a mark at baseline minus one second.
  - id: F2
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/orchengine/watcher.go:1233
    summary: The /clear reload recording PointerMarkSince as the unchanged baseline has no test; removing the assignment leaves the field zero and no row fails
    fix: Add a test in watcher_test.go driving a clear cycle with a set CompactionBaseline, a pending resume pointer and a same-text resume mark older than that baseline; expect the pointer typed and PointerMarkSince zero after idle.
  - id: F3
    severity: LOW
    class: scope
    origin: lens:code-affected-set
    location: internal/orchengine/state_test.go:315
    summary: ResetForFreshLaunch zeroing PointerMarkSince (decision D2) has no assertion; deleting state.go:464 breaks no test
    fix: In TestResetForFreshLaunch's non-idle input set PointerMarkSince to a non-zero time and assert got.PointerMarkSince.IsZero() beside the existing cleared-field assertions. Add no new test function.
---

### [LOW] Requested compact cycle's previous-baseline recording is unpinned

**Location:** internal/orchengine/watcher_compact_test.go:501, internal/orchengine/watcher.go:1649
**Issue:** The new compact-cycle test passes with `PointerMarkSince` left zero, since a zero field accepts any matching mark and the test's baseline is unset. Only the automatic-path row "a mark older than the previous baseline types the pointer" pins the lower bound, so the `tickCompacting` assignment can be removed unseen. The acceptance criterion on a mark older than the previous baseline is met for the automatic path only.

### [LOW] The /clear path's baseline carry-over is untested

**Location:** internal/orchengine/watcher.go:1233
**Issue:** Decision D1 states that the reload after `/clear` records the baseline unchanged. No test runs a clear cycle to the pointer step with a stale same-text mark. Dropping the assignment would make the zero field accept a stale mark and skip the pointer.

### [LOW] Fresh-launch zeroing of PointerMarkSince is untested

**Location:** internal/orchengine/state_test.go:315
**Issue:** `ResetForFreshLaunch` zeroes the field and its doc comment says so, per D2, but neither input of `TestResetForFreshLaunch` sets the field or asserts on it.

## Rejected

- Persistence across a watcher restart lacks a dedicated test (lens:code-tests, item 6): every set precedes `enter`, which saves, and the tests read the state through the persisted file, so a dedicated restart test would cover only `State`'s existing JSON path.
- No test decodes a state file without `pointer_mark_since` (lens:code-tests, item 9): decoding a missing key as zero is encoding/json behavior shared by every State field, so a test would pin the standard library.
- Other `toIdle` exits (timeout, withdrawn) lack their own zeroing assertion (lens:code-tests, item 5): all exits share the single `toIdle` line that the skip rows already pin.

The code-correctness lens found no defects, and the holistic read agrees: the field is set before the baseline moves in all three reload paths, persisted by `enter` before the pointer step can run, and zeroed by `toIdle` and `ResetForFreshLaunch`. The diff stays inside the record's scope, and the package doc and State field doc describe the new check accurately.
