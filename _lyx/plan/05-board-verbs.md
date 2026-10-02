# Card 5 — board-verbs

**Edit:**
- `internal/boardengine/board.go`
- `internal/boardengine/board_test.go`
- `internal/boardengine/sync_integration_test.go`

**Uses:**
- `internal/boardengine/store.go`
- `internal/boardengine/legacy.go`

**Intent:**
Expose the new operations on the `Board` facade, each write through `boardCriticalSection` (Decisions 10, 14–16).

- `Promote(slug string, target *int) (Task, error)` runs `Store.Promote` under the lock.
- `Prune() ([]string, error)` runs `Store.Prune` under the lock and returns the removed slugs; the render that follows drops their design docs through the sidecar.
- `Find(text string) ([]BriefTask, error)` is a read: it loads the store without the lock and returns `Store.Find`.
- `RetireLegacy() error`: under the lock, refuses with "nothing to retire" when neither legacy file exists;
  otherwise loads (which runs the last fold), drops `legacy_done`, saves `board.json`, deletes `tasks.json`, `notes.json` and their `.swaplock` files, renders, and spawns sync.
  `board.json` is saved before any deletion, so a crash in between leaves the legacy files beside a store that has already absorbed them.

Tests in `board_test.go`: promote through the facade persists the new tier; prune removes a `done` entry and its design doc from disk and keeps an `abandoned` one;
find returns matches from an unmigrated board without writing;
retire-legacy on a board with both legacy files folds a pending done mark, leaves a `board.json` without `legacy_done`, and removes both legacy files and their swaplocks;
retire-legacy on a board with no legacy file refuses and writes nothing.

`sync_integration_test.go`: after `RetireLegacy` on a synced board that tracked the legacy files, `Sync` commits `board.json` and the legacy-file deletions, leaving a clean tree.

**ImpactSummary:** Additive facade methods; no existing caller changes.
