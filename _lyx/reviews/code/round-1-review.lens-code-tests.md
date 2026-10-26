```yaml
lens: code-tests
outcome: reported
```

**Per behaviour: the test that pins it**

1. **Plain row skipped and never visited is boxed `[-]` with no time and no visit count.**
   - Pinned by the `TestRenderChecklist` row at `checklist_test.go:319-331`, the rewrite of the old "skipped row is passed through" row.
   - Not a duplicate. It rewrites the old row, which asserted `[ ]`, and no other row covers the skipped box for a plain row.

2. **A row both skipped and visited is boxed `[x]`.**
   - Pinned by the new row at `checklist_test.go:332-343`.
   - It would fail if the skipped check came before the visit check in `checklistBox`.
   - Not a duplicate. It is the only row that mixes skip entries and visits.

3. **The halted box moves from `[-]` to `[=]`.**
   - Pinned by the existing rows whose expectations were edited, at lines 389, 401, 413, 425, 436 and 875.
   - These cover awaiting, blocked and the segment heading. The halted box has no new row, which is right because the existing rows already assert it.

4. **A review segment that was never visited but has a skipped row gets a single `[-]` heading line.**
   - Pinned by the new row at `checklist_test.go:901-914`, which skips only `Rev-Bouncer`.
   - Not a duplicate. The segment-level `[ ]` rows at lines 611-694 and 894 have no skip entries.

5. **An unreached row or segment is still `[ ]`.**
   - Pinned by those same rows, which stay unchanged: `[ ] 3. Build` in the halted rows, and `[ ] 2. Rev` in the rows at lines 611-694 and 894.

6. **`checklistSkippedRows` reads only `OutcomeSkipped` history entries.**
   - Covered through the public `renderChecklist` rows above, so no per-helper test is needed.
   - No row puts a non-skipped outcome next to a skipped one for the same producer, but behaviours 1 and 2 would break if the filter were dropped.

**Uncovered behaviours**

- **A visited review segment's heading never takes `[-]` from a skipped row inside it.**
  - This is the explicit `false` argument and its comment at `checklist.go:427-428`. I found no row that pins it.
  - The rows with a skip in a review segment (line 905) are all unvisited. The visited-segment rows (lines 875 and 894) have no skip entry.
  - Evidence is the grep over `checklist_test.go`, where `skippedEntry` is used only at lines 322, 335 and 905.
  - If `false` were changed to `skipped[row.Name]`, the heading could pick up `[-]` from an unvisited skipped row, since `[-]` is neither done nor not-reached. No test would catch that.
  - Severity LOW, class `scope`, location `internal/loomcli/checklist_test.go` (a row beside line 905).
  - A row with a visited segment that also has a skipped row, expecting heading `[x]`, would pin it.

- **A current producer that also has skip entries keeps its state box.**
  - This is the precedence "current producer, then skipped" in `checklistBox`, with `checklistBox` returning the state box first.
  - No row has a running or halted current producer with a skip entry.
  - Severity NIT, class `scope`, location `internal/loomcli/checklist_test.go`.
  - The `[=]` and `[>]` rows pin the state box itself, so I report it only as a precedence gap.

**Test economy**

- No new test duplicates existing coverage.
- No per-helper test was added.
- No review-fix test was added.
- No assertion on a stencil's static prose was added.
- The doc.go paragraph change needs no test, because it is documentation.

**Tool calls**

1. `grep` over `internal/loomcli/*_test.go` for `OutcomeSkipped`, `skippedEntry`, `[ ] N. Rev`, `[-]` and `[=]`, because my context did not show which other test rows assert those boxes or use skip entries.
