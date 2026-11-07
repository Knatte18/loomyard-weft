```yaml
lens: code-tests
outcome: reported
```

**Per behaviour: the test that pins it, or none**

1. **Design doc carries the `Decision record:` line under the metadata line and before `Depends on:` for a discussed entry.**
   - Pinned by `TestRenderDesignDocGoldens`, new `design-discussed.md` row (`render_test.go:1310-1316`). It is an exact-string golden with a dependency, so it fixes both the text and the position.

2. **A non-discussed entry's design doc has no record line.**
   - Pinned by the existing `main` and `plain` rows of `TestRenderDesignDocGoldens`, which are exact-string and carry no such line. No new test is needed.

3. **`RenderToDisk` unflags a discussed entry whose record file is missing, so neither the README row nor the design doc links it.**
   - Pinned by `TestRenderToDiskLinksOnlyAStoredDecisionRecord`, rows `MissingRecordReadmeRow` and `MissingRecordDesignDoc` (`render_test.go:1155,1157`). Both would fail if `withMissingRecordsUndiscussed` were removed.

4. **`RenderToDisk` keeps the link when the record file exists, in both the README row and the design doc.**
   - Pinned by rows `StoredRecordReadmeRow` and `StoredRecordDesignDoc` of the same test (`render_test.go:1154,1156`).

5. **`withMissingRecordsUndiscussed` copies the entries and does not mutate the caller's slice.**
   - None. Nothing asserts that `tasks[i].Discussed` is unchanged after `RenderToDisk`.
   - This matters because `renderOwnerWrite` (`ownerset.go:246-251`) and `boardCriticalSection` (`board.go:197-205`) keep using the same slice afterwards, and `store.Save` runs earlier.
   - Severity LOW, class `design`. Suggested fix: one assertion in the existing `RenderToDisk` test that `tasks[1].Discussed` is still true after the call.

6. **A Stat error other than not-exist (permission, I/O) is treated as a missing record.**
   - None, and awkward to test hermetically. This is the focus directive's question.
   - Severity LOW at most, class `consistency`. It depends on whether the handler accepts that reading of "missing".

7. **A discussed entry with an empty body gets no design doc, and the README link is the only route.**
   - Pinned for `Render` alone by `TestRenderSlugCellLinksTheDecisionRecord`, entry T3, which has no body and a Sketch link in `render_test.go:1111`.
   - Not pinned through `RenderToDisk`, where the existence check also applies. This is an unpinned branch rather than a gap, since the code has no body-dependent path in the check. NIT.

8. **The behaviour holds for any kind, for a promoted task and for a raddle part.**
   - There is no kind-dependent code. The new `RenderToDisk` test and the golden both use `KindTask`, and `TestRenderSketchSection` covers the note row.
   - Not a gap, since the flag alone decides the rendering.

9. **The board write, owner write and Bolt replay paths write the record file before they render, so a freshly stored record is linked in the same write.**
   - None at the level of an end-to-end write. I found no test outside `render_test.go` that asserts `Decision record:` or `[Sketch]`.
   - The ordering is a claim in `summary.md` ("each writes the record files before that render") that nothing pins.
   - Severity LOW to MEDIUM, class `design`. A regression would be silent: a flagged entry unlinked until the next render.
   - Suggested fix: a tier0 test that calls `Board.StoreDecisionRecord` on a board and then reads `README.md` for the link.

**Test economy**
- `TestRenderToDiskLinksOnlyAStoredDecisionRecord` covers disk existence, which no existing test covers, so it is not a duplicate of `TestRenderSlugCellLinksTheDecisionRecord`.
- The tier0 rules hold: `tiergate.Require(t, 0)`, `t.Parallel`, `t.TempDir`, and no spawn or sleep.
- The README-row rows use the string `"| T1 | **Stored** | [`stored`](design-stored.md)<br>[Sketch](decision-stored.md) | darn |"`, which duplicates the row format that `TestRenderSlugCellLinksTheDecisionRecord` already pins. This is a minor overlap and not a finding.

**Tool calls (1):**
1. A grep of `*_test.go` for `decision-`, `StoreDecisionRecord`, `Decision record:` and `[Sketch]`, to find any end-to-end test of the write paths that links a record.
