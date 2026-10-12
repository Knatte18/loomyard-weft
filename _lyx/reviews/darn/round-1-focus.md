---
round: 1
focus:
  - "internal/boardengine/dates.go and board.go: does stampDates run on every write path (upsert, batch, merge, set-deps, promote, set-status, intake) and compare against entries loaded under the lock, so a status-only change, a prune of a done dependency and a note-to-task promote never bump revised?"
  - "internal/boardengine/runlock.go: does scopeChanged ignore only created and revised, and does a backfill upsert on a run-held entry pass while any real scope change is still refused?"
  - "internal/boardengine/store.go and task.go: do entries without dates load unchanged, and is a missing date omitted from board.json rather than written as a zero time?"
  - "merge handling: does the upserted entry get the oldest known created among itself and the removed entries, ignoring unknown dates?"
  - "internal/boardengine/render.go and FormatAge: unit boundaries (45m, 5h, 3d, 5w, 4mo, 2y), blank output for unknown dates, and the render-time line; are all Render and RenderToDisk callers updated?"
  - "internal/boardcli/cli.go and text.go: --sort age and --sort revised ordering (oldest first, unknown last, ties keep README order, other values refused), and column dropping when no listed entry has the date."
  - "Docs and tests against the repository rules: doc.go, help text and the ly:board skill match the behavior; the test-budget edit matches the new top-level tests; TestEntryDates and the extended goldens do not duplicate existing coverage and every added behavior (sorting, FormatAge, clock injection) has a test."
  - "_lyx/landing/summary.md claims against the diff, and scope against the board entry: nothing beyond the entry is made, and the backfill stays outside the diff."
---

Seed focus from the summary and diff stat; the reviewer still reads the whole diff.
