# Card 9 — merge carries the removed entries' issues

**Edit:**
- `internal/boardengine#Store.MergeTasks`
- `internal/boardengine/doc.go`
- `internal/boardengine/store_test.go`
- `internal/boardcli/doc.go`
- `plugins/ly/skills/board/SKILL.md`

**Intent:**
Implement D7.
`MergeTasks` computes the upserted entry after the patch, then sets its `issues` to that entry's own `issues` followed by each removed entry's `issues`, in `remove_slugs` order, dropping duplicates and keeping first occurrence.
It writes the final list into a copy of the `upsert` map under `issues` before `validateWrite` and `UpsertTask` run, so the validated entry and the saved one are the same.
It applies whether or not the payload names `issues`, with no flag to turn it off (A6).

Bound: a merge only adds issue numbers that a removed entry recorded, and never drops one the payload or the existing entry holds.
A carried number that an untouched third entry also records is carried as-is: the merge adds no cross-entry uniqueness check.
A `remove_slugs` name that matches nothing contributes nothing.

Tests, untagged: removing entries with issues `[3]` and `[3, 5]` into an upsert holding `[1]` yields `[1, 3, 5]`; a payload naming `issues` keeps its own numbers first; a remove slug that matches nothing adds nothing; a set-status failure still leaves the store unchanged.

`doc.go` (both packages) and `SKILL.md` state that a merge carries the removed entries' issues.

**ImpactSummary:** every `lyx board merge` whose removed entries record issues; merges without issues are unchanged.

**Commit:** 9: merge carries the removed entries' issues
