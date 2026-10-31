```yaml
lens: code-tests
outcome: reported
```

**Per behaviour: the test that pins it**

1. **Automatic compaction: a mark stamped before the boundary but after the previous baseline skips the pointer.**
   - Pinned by `TestWatcher_HookDeliveredPointerIsSkippedAndAnythingElseIsTyped`, row "a mark the hook stamped before the boundary was written skips it" (`watcher_test.go` ~1476).
   - It sets the baseline to now−1min and the mark to boundary−30ms. If `startAutoReload` stopped recording `PointerMarkSince`, the field would be zero and the row would still pass, so the row cannot catch that. The old `CompactionBaseline` comparison is what it catches: the mark sits before the new baseline, so the old check fails the row.

2. **Requested compact cycle: the same skip.**
   - Pinned by `TestCompact_HookMarkStampedBeforeTheBoundaryIsNotTypedAgain` (`watcher_compact_test.go:501`).
   - It fails under the old comparison, since the mark is 30ms before the boundary. It does not distinguish "bound recorded from the previous baseline" from "field left zero", because `newCompactEnv`'s baseline is not set in the test (grep shows no `CompactionBaseline` set in `watcher_compact_test.go` outside line 172). That is acceptable.

3. **A mark older than the previous baseline types the pointer.**
   - Pinned by the row "a mark older than the previous baseline types the pointer" (`watcher_test.go` ~1483).
   - This is the only test that fails if `PointerMarkSince` were recorded as zero (the field-zero case would accept the old mark). It covers the automatic path only. No test covers it for the requested compact cycle.

4. **A mark with other text types the pointer.**
   - Pinned by the existing row "a mark with other text types the pointer" (`watcher_test.go`). It is unchanged by the diff and still valid.

5. **D2: the field is zero after the reload returns to idle.**
   - Pinned by the `!st.PointerMarkSince.IsZero()` assertion in both tests above, for the skip-ends-in-idle path through `toIdle`.
   - The rows that type the pointer do not reach idle through `toIdle` in that test, so only the skipping rows exercise the zeroing.
   - **No test covers other `toIdle` exits** (timeout, withdrawn, abort). They all share the same `toIdle` line, so this is a minor gap.

6. **The field is persisted (D2: restart mid-reload reads the same bound).**
   - **No test.** Nothing stops and restarts the watcher between `startReload` and the pointer step with a non-zero field. The `enter` save carries it, but no row would fail if the field lost its json tag or were dropped from the save.
   - Weak coverage rather than a missing behaviour, since the tests that read `e.state()` read through the persisted file.

7. **`ResetForFreshLaunch` zeroes `PointerMarkSince`.**
   - **None.** `TestResetForFreshLaunch` ("non-idle", `state_test.go` ~315) sets no `PointerMarkSince` in its input and asserts nothing about it. Deleting `s.PointerMarkSince = time.Time{}` (`state.go:464`) breaks no test.

8. **`/clear` path (`tickClearing`): the field takes the baseline unchanged, and `hookDelivered` behaves as before.**
   - **None.** No test sets a baseline and a mark in a clearing cycle and checks the pointer step's skip or type decision. Removing `st.PointerMarkSince = st.CompactionBaseline` at `watcher.go:1233` leaves the field zero, which accepts every matching mark, so a stale same-text mark would be wrongly skipped, and no row fails.

9. **A zero field (older state file) accepts any matching-text mark (D1 Bound).**
   - **None** as an explicit case. No test decodes a state without `pointer_mark_since`, or holds a zero field with a matching old mark.
   - The only zero-field path, a legacy file, is untested.

10. **`hookDelivered` reads `PointerMarkSince`, not `CompactionBaseline`.**
    - Pinned by rows 1 and 2, as in item 1.

**Findings**

- **F1, LOW, design (test economy: uncovered behaviour):** `ResetForFreshLaunch` clearing `PointerMarkSince` has no assertion.
  - Evidence: `state.go:464`; `state_test.go` `TestResetForFreshLaunch` non-idle input.
  - D2 names the fresh launch as a clearing point.
  - Fix: add the field to the non-idle input and one assertion, not a new test.
- **F2, LOW, design (uncovered behaviour):** the `/clear` reload recording `PointerMarkSince = CompactionBaseline` (`watcher.go:1233`) has no test.
  - Nothing fails if that line is removed or if the `/clear` path took the post-move baseline.
  - D1 states this as an explicit decision.
  - Fix: add a row for the clearing path with a stale same-text mark older than the baseline, expecting the pointer typed.
- **F3, LOW, design (uncovered behaviour):** "mark older than the previous baseline types the pointer" is pinned only on the automatic path.
  - The requested compact path (`watcher_compact_test.go:501`) has only the positive skip case.
  - Removing `st.PointerMarkSince = st.CompactionBaseline` at `watcher.go:1649` breaks no test, because the field then stays zero and the existing compact test's mark still matches.
  - Acceptance criterion D1 lists the requested cycle only for the skip case, so this is a gap in recording the previous baseline, not in the criteria.
- **No findings on test duplication:** the new compact-cycle test exercises a different path (`tickCompacting`) from the automatic one, so the two do not duplicate.

**Tool calls:** 1. grep `CompactionBaseline` in the watcher test files and list the files mentioning `PhaseClearing`; my context lacked whether the compact test sets a baseline and whether a clearing test exists.
