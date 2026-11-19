# Card 2 — entry-revision-check

**Create:**
- `internal/boardengine#StaleEntryError`
- `internal/boardengine#StaleEntryError.Error`
- `internal/boardengine#Board.Expecting`
- `internal/boardengine#Board.checkExpected`
- `internal/boardengine_test#TestExpectingRevision`

**Edit:**
- `internal/boardengine#Board`
- `internal/boardengine#Board.boardCriticalSection`

**Extends:**
- `internal/boardengine/doc.go`

**Uses:**
- `internal/boardengine#buildView`
- `internal/boardengine#EntryRevision`
- `internal/boardengine#migrateLoaded`
- `internal/boardengine#Board.View`
- `internal/boardengine#withMissingRecordsUndiscussed`
- `internal/boardengine#Board.UpsertTaskReporting`
- `internal/boardengine#Board.SetStatusReleasingProject`

**Intent:**
Check an entry's expected revision inside the board's write section (D8).

- `Board` gains an unexported `expected` field: nil, or one slug with the revision a write expects.
  `Expecting(slug, revision)` returns a shallow copy of the board carrying it; the receiver is not changed, so the CLI's board and every verb behave as today, with no revision.
  An empty revision means the write expects no entry of that slug: a create.
- `boardCriticalSection` calls `checkExpected` right after `migrateLoaded` and before `fn` runs, under the write lock, on the entries as loaded and migrated.
  A nil `expected` passes.
  It refuses with a `*StaleEntryError` and returns before the mutation, so nothing is saved, rendered or synced:
  - a non-empty revision whose slug is absent: `Fresh` is nil, message `entry "<slug>" was removed since the page read it, so nothing was written: way forward: reload the board and act on what it holds now`;
  - a non-empty revision unequal to `EntryRevision` of the loaded entry: `Fresh` is that entry's projection, message `entry "<slug>" changed since the page read it, so nothing was written: way forward: compare the fresh entry and send the edit again`;
  - an empty revision whose slug exists: `Fresh` is the existing entry's projection, message `entry "<slug>" already exists, so nothing was written: way forward: pick another slug, or edit the existing entry`.
  The projection is the entry of `buildView(loaded, withMissingRecordsUndiscussed(b.boardPath, loaded), primeOwner, b.out, b.now())`, the function `Board.View` uses, so the fresh entry matches what a reload shows.
- A slug refused for any other reason, such as one a raddle store holds, is refused by the existing checks with the board's own message, unchanged.
- The check covers only the entry `expected` names: a sibling write the board makes as a side effect, such as a project released with its last member, is not checked, as the CLI's same write is not.
- `doc.go`'s write-section paragraph gains the revision check and its three refusals.

Bound: the check adds a refusal and lets nothing new through; a board without `expected` takes the exact path it takes today.

`TestExpectingRevision`, a new tier0 table test in `internal/boardengine_test`, builds a `Board` over a temp board with `SkipGit` set and reaches every branch through the board's exported writes:
- an edit through `Expecting(slug, <revision Board.View gave>)` and `UpsertTaskReporting` is saved;
- an edit carrying an older revision, including one older only by a status change made through `SetStatusReleasingProject`, is refused with a `*StaleEntryError` whose `Fresh` revision is the current one, and every file of the board directory, the render manifest included, keeps its bytes and no file is added, so the refusal leaves the Bolt's tree as the last commit's sync left it;
- on a `board.json` with no `draft_status` key holding a task with an empty status, the revision `Board.View` gives that entry is accepted by an edit;
- an edit of an entry removed by `RemoveTask` is refused as stale with a nil `Fresh`;
- a create through `Expecting(slug, "")` of an existing slug is refused with the existing entry's projection, and of a new slug is saved.
The existing CLI write tests, `TestUpsertTask` among them, keep pinning that a board without `Expecting` writes as today.

**ImpactSummary:** one new unexported Board field read only inside boardCriticalSection; every existing caller builds boards without it, so the CLI and every other writer are unchanged.

**Commit:** 2: board: refuse a write whose entry revision is stale
