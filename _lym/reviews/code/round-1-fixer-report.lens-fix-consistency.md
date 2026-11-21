```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the change set against its surroundings.

**Findings**

1. **MEDIUM** — `internal/fabricengine/mergeguards.go:286` (`recordedMergeGoneReason`, weft twin of the changed `sideRecordedMergeGone`).
   - Summary: F3 removed the weft conclude arm and the empty-source exemption, but the guard still evaluates the weft side.
   - Evidence:
     - `sideRecordedMergeGone(f.records, ..., st.WeftOutcome, st.WeftSource, ...)` still runs.
     - Any record whose weft outcome is `staged` or `conflicted` has an empty `WeftSource` (the review's older-binary case), and the guard no longer exempts that.
     - It now reaches `repo.IsAncestor("", head)` and returns an "invalid SHA" error, the failure `TestMergeCrucible_ContinueRefusesAttemptThatNeverReachedBothSides` showed before the fixer edited that test.
     - `concludeMergeSides` no longer concludes the weft, so the weft guard now only produces an error or a refusal for a side nothing concludes.
   - Fix direction: drop the weft call from `recordedMergeGoneReason`, as the fixer did for `concludeMergeSides`.

2. **LOW** — `internal/fabricengine/doc.go:2013`.
   - Summary: the text "The arm also serves the legacy-record case `concludeMergeSides` handles" still describes the deleted weft arm.
   - Evidence: it is the only remaining reference to the legacy-record case after the `concludeMergeSides` weft arm was removed. Lines 2142-2150 (the `concludeLandedReason` text about `staged`/`conflicted` sides) also still speak of both sides.

3. **LOW** — `internal/fabricengine/mergestate.go:42-45, 105`.
   - Summary: the `WeftSource` doc ("legitimately empty when no counterpart resolves") now sits oddly beside the removed exemption.
   - Evidence: the comments say the weft side is always `up_to_date`. They agree with finding 1's fix but not with the guard code that still reads `WeftSource` and `WeftOutcome` for `staged`.

No other stale references: greps for `ScopeWasPartial`, `migrateRunSettings`, `Bolt legacy`, `git mv .fabric-anchor` and "reads a stored partial" return nothing outside research and benchmark reports.

**Tool calls**
- `grep` over `.go` and `.md` for the removed-concept names, plus `mergestate.go` lines: my context lacked a repo-wide twin search.
- `grep -n "func recordedMergeGoneReason"` in `mergeguards.go`, plus `grep` of `concludeMergeSides` in the fabric `doc.go`: my context showed only the `sideRecordedMergeGone` span, not its weft caller.
