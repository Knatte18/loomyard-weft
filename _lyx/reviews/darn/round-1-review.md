---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/boardengine/dates.go:44
    summary: stampDates exempts every note-to-task kind change from revision, so a plain upsert of kind task on a note also keeps revised, though the entry counts a kind change as a scope revision and only promote keeps the note's dates
    fix: In internal/boardengine/board.go make Promote's closure return a distinct marker (for example a promoted type wrapping the Task) and have boardCriticalSection pass the promoted slug to stampDates as an exemption; delete the before.Kind == KindNote && judged.Kind == KindTask rewrite in stampDates (dates.go:44-46) so any other write that flips kind is a revision. Add a row to TestEntryDates (board_test.go) that upserts kind task on a note and expects revised to move.
  - id: F2
    severity: LOW
    class: consistency
    location: internal/boardengine/dates.go:56
    summary: FormatAge has no test of its unit boundaries or its clamp, and the minutes unit is only reached through the integration-tagged 0m listing
    fix: Add a table-driven TestFormatAge in internal/boardengine (a new file dates_test.go or in render_test.go) with rows at 0 and 45m, 5h, 13d and 14d, 90d and 91d, 364d and 365d, 2y, a zero since giving the empty string, and a since after now giving 0m; raise the internal/boardengine row in cmd/lyx/testdata/test-budget.yaml by one and list that file under Edit.
---

### [LOW] Every note-to-task kind change keeps revised, not only promote

**Location:** internal/boardengine/dates.go:44
**Issue:** The board entry says a change to kind is a revision and that a promote keeps the note's dates. `stampDates` rewrites any note-to-task kind difference away before `scopeChanged`, and it cannot tell `Promote` from an `upsert` that sets `kind` to `task`. A pure kind flip through `upsert` therefore does not bump `revised`, contrary to the entry. A flip that also changes another scope field still revises, so the gap is narrow.

### [LOW] FormatAge boundaries and clamp are untested

**Location:** internal/boardengine/dates.go:56
**Issue:** The diff adds unit boundaries (hour, day, 14 days to weeks, 91 days to months, 365 days to years), a clamp of a future date to `0m`, and a blank for an unknown date. The README goldens and `TestRenderCompact` reach `3d`, `5h`, `5w`, `5mo` and `1y` and the blank. The boundary values, the `m` unit and the clamp are reached only by the integration-tagged `list --text` step, which shows `0m` for entries created in the same run. No existing test pins the boundaries, so a shifted threshold passes every test.

## Notes

The focus questions were checked, and none produced a finding beyond the two above.

- `stampDates` runs inside `boardCriticalSection` on every write path against entries cloned under the lock. The status-only, prune-of-done-dependency and promote cases are exempted, and `TestEntryDates` pins status change, promote, backfill on a run-held entry and merge.
- `scopeChanged` ignores only `Created` and `Revised`, since it never compared them and lists the other fields explicitly. A real scope change on a run-held entry is still refused.
- Entries without dates load unchanged, and `omitzero` omits an unknown date from `board.json` and from `BriefTask`.
- A merge takes the oldest known `created` among the upserted entry and the removed entries, ignoring unknown dates.
- Every `Render`, `RenderToDisk` and `RenderCompact` caller is updated, including the benchmark. `go test` passes for `internal/boardengine` and `internal/boardcli`.
- `--sort age` and `--sort revised` order oldest first with unknown last and stable ties, and any other value is refused. The integration step covers the three cases.
- The test-budget bump of one matches the single new top-level test, and the docs, help text and skill match the behavior. The summary's claims are borne out by the diff, and the backfill is outside it.
