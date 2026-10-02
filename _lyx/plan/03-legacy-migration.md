# Card 3 — legacy-migration

**Create:**
- `internal/boardengine/legacy.go`
- `internal/boardengine/legacy_test.go`

**Uses:**
- `internal/boardengine/task.go`

**Intent:**
Hold the conversion from the pre-upgrade `tasks.json` + `notes.json` records into store entries, and the done-mark fold, as pure functions over decoded slices, so card 4's I/O layer stays thin and every migration rule is unit-testable without a disk (Decisions 8–9).

- A legacy record type matching the pre-upgrade on-disk shape: `id`, `slug`, `title`, `depends_on`, `isolated`, `deferred`, `brief`, `body`, `status`, `type` (the recipe), `short_name`.
  It is decoded leniently, the way the old binary read its own files.
- The legacy file names `tasks.json` and `notes.json` are declared here, as constants card 4's I/O and card 5's `RetireLegacy` both read.
- Migration takes the decoded tasks and notes and returns the entries plus the seeded `legacy_done` slug list:
  tasks become tier 1, deferred tasks tier 3, notes tier 3;
  every entry gets `type: feature`; a legacy `type` value moves to `recipe`; ids, `depends_on`, `isolated`, `brief`, `body`, `status` and `short_name` carry over.
  A note whose id collides with a task id gets a fresh id above every id already taken.
  A slug present in both legacy files is an error naming the slug and saying the duplicate must be removed from one legacy file.
  `legacy_done` is seeded with every legacy slug whose status is `done`.
- The fold takes the store entries, the store's `legacy_done`, and the decoded legacy records, and returns the folded entries and the grown `legacy_done`:
  for each legacy record with status `done` whose slug is not in `legacy_done`, the store entry with that slug is set `done` when one exists, and the slug joins `legacy_done` either way.
  No other status, no new slug, and no other field is ever folded.

Tests in `legacy_test.go`:
- Migration: tier and type assignment for a task, a deferred task and a note; `type` moved to `recipe`; ids preserved; a colliding note id reassigned to a fresh one; a slug in both files refused naming it; `legacy_done` seeded with exactly the done slugs.
- Fold: a new legacy `done` mark applies once and joins `legacy_done`; a mark for a slug absent from the store is ignored but recorded;
  a non-done legacy status changes nothing; an entry `done` at migration and later reopened stays reopened across repeated folds;
  a new entry reusing a slug already `done` in the legacy file is not marked `done`.
