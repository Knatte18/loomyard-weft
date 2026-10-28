```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, listing for each behaviour the diff adds or changes the test that pins it, or none.

**Per behaviour: pinning test**

1. The gate card ID is not read as a weft sibling.
   - Pinned by the `TestRefScanner_Matches` row "gate card ID ending in the suffix is text" (`refscanner_test.go:68`).
2. The same, chained after `git commit -m "…" &&`, with a quoted multi-line message.
   - Pinned by the row "gate card ID after a commit with a quoted message is text" (`refscanner_test.go:69`).
3. A separator-prefixed weft path inside a gate segment still matches.
   - Pinned by the row "gate test with a separator-prefixed weft path matches" (`refscanner_test.go:70`).
4. A bare weft name in a later segment still matches, so blanking stops at a separator.
   - Pinned by the row "bare weft name after a gate segment matches" (`refscanner_test.go:71`).
   - It exercises `&&` only.
5. The gate blanking stops at each of `;`, `||`, `|` and newline.
   - Pinned by none.
   - The `|` in row 1 is followed by `tail -40`, which holds no weft name, so the row would still pass if `|` did not end the segment.
6. The blanking applies only in command position.
   - Pinned by none.
   - There is no row for `lym gate` appearing as an argument, such as `echo lym gate other-weft`, which must still match.
7. Wrapper and prefix forms (`env`, `VAR=x`, a path-prefixed `lym`, `lym.exe`, `$(`, backtick, `(`) are recognised in command position.
   - Pinned by none.
8. The worktree's exact sibling path inside a gate segment still matches.
   - Pinned by none in the diff.
   - This is the D1 bound, and I have not checked the `RefScanner` tests for a gate-segment row.
9. `recover-batch` with no recorded start names why accept-audit refuses, and names no accept-audit route.
   - Pinned by `noStartCheck` on `fabricRefusalCheck`, and by the row "a pathless entry longer than NAME_MAX alone with no recorded start names why accept-audit refuses".
   - It asserts the `noStartText` substring and the absence of `acceptBatchStep`.
10. `recover-batch` at HEAD equal to the start on a clean tree names accept-audit then recover-batch.
    - Pinned by the row "a mutating fabric reference at the batch's start on a clean tree is refused toward the accept-audit route".
    - It asserts the exact `way forward: …; otherwise 1) lym webster reset --to start; 2) lym webster run`.
11. `recover-batch` at HEAD past the start with a mutating reference names `reset --to batch-start`, then accept-audit and recover-batch.
    - Pinned by the row "a mutating fabric reference past the batch's start names the batch-start reset before accept-audit".
    - It asserts `ErrRecoveryNeedsFresh`, no spawn, and the exact suffix.
12. `acceptAuditWayForward` returns a non-`ErrAuditNotAcceptable` error, a git read failure, instead of a refusal.
    - Pinned by none.
13. The refusal on a dirty worktree, or on HEAD not descending from the start, now carries the `rerun` text "re-run accept-audit then recover-batch".
    - Pinned by none through `recover-batch`.
    - Neither branch has a row, so the `Contains(stepResetToStart)` append logic is untested for them.
    - The dirty-tree text is probably still covered through the `accept-audit` tests.
14. `batchFabricReferenceEvidence` treats a nil `readOnly` as accepting nothing, a guard added for `recover-batch`.
    - Pinned by none.
    - The `recover-batch` rows set `ReadOnly` explicitly, and `fabricRefusalBase` is not shown to leave it nil.
15. `accept-audit`'s own refusals and its acceptance are unchanged after the refactor.
    - The pinning tests are existing `AcceptBatchFabricReference` tests, which I did not read.
    - Messages with `verb` now come from the `rerun` string `re-run "lym webster %s %d"`, which gives the same text.
    - No new test is needed.
16. `PlanReset` under a held run lock names the parent re-running the reset after the run stops, and `report-head` still passes the lock.
    - Pinned by `TestPlanReset_HeldRunLockIsTransientBusy`, which now uses `batch-start` with batch 6 and asserts the exact suffix.
    - The `report-head` pass-through assertion is kept.

**Findings**

- **[LOW] [design]** `internal/fabricengine/refscanner_test.go:68-71`. No row pins the D1 bound that the blanking applies only in command position, or that it stops at `;`, `||`, `|` and newline.
  - A regression that blanked `echo lym gate x-weft`, or let the segment run past `|`, `;` or a newline, would pass every row.
  - Add rows to the same table: `echo lym gate other-weft` (true), `lym gate test --card a-weft; ls other-weft` (true), and a `|`-separated case with a weft name after the pipe (true).
  - Evidence: `refscanner.go:29-34`, which makes the segment's end and its start condition load-bearing.

- **[LOW] [design]** `internal/websterengine/recoverbatch_test.go:~1425-1455`. `acceptAuditWayForward` has no row for the other refusal branches or for a git failure.
  - The branches with no row are a dirty tree, a HEAD that does not descend from the start, and a git read error that must be returned rather than turned into a refusal.
  - The no-start, at-start and past-start rows do not reach them.
  - Add rows to `TestRecoverSpawnOrAttach`.
  - Evidence: `recoverbatch.go:574-594`, `auditledger.go:253-258`.

- **[NIT] [design]** `internal/websterengine/auditledger.go:263`. The new `readOnly == nil` guard has no row.
  - Either drop the guard, since `recover-batch` is the only nil caller and the fixture sets `ReadOnly`, or add a row that leaves `Deps.ReadOnly` nil and expects the batch-start route.
  - The decision record does not ask for the guard.

**Tool calls:** none. The diff and the manifest in context were enough.
