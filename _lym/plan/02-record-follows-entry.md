# Card 2 — record-follows-entry

**Create:**
- `plan:internal/boardengine#followDecisionRecords` -> `func followDecisionRecords(boardPath string, loaded, after []Task, renamedFrom, renamedTo string) error`
- `plan:internal/boardengine_test#TestDecisionRecordFollowsEntry` -> `func TestDecisionRecordFollowsEntry(t *testing.T)`

**Edit:**
- `internal/boardengine#Board.boardCriticalSection`
- `internal/boardengine#Board.Rename`
- `internal/boardengine/doc.go`

**Uses:**
- `plan:internal/boardengine#decisionFileName`
- `plan:internal/boardengine#Board.StoreDecisionRecord`
- `plan:internal/boardengine#Board.DecisionRecord`
- `internal/logger#`
- `internal/boardengine#Board.RemoveTask`
- `internal/boardengine#Board.Prune`
- `internal/boardengine#Board.MergeTasks`
- `internal/boardengine_test#newRenameBoard`

**Intent:**
Make the record file follow its entry through a rename and a removal (D4).

`boardCriticalSection` gains one step after the save and `afterSave`, and before the render, under the board lock.
It calls `followDecisionRecords(b.boardPath, loaded, store.Tasks(), from, renamedTo)`.
`loaded` here is the entry list as the run lock judges it, with the rename already applied.
`from` is the `RenameResult`'s `From`, and is empty for any other write.

The step does two things:
- On a rename, it moves `decisionFileName(from)` to `decisionFileName(renamedTo)` with `os.Rename`. Any failure is returned wrapped: `rename moved the entry but not its decision record: %w: the record stays at %s until the entry's next Discussion approval stores it under the new slug`.
- When `decisionFileName(from)` is absent, a flagged entry with no file, it removes `decisionFileName(renamedTo)` if that file exists, so the renamed entry reads as flagged without a file and never adopts an orphan left at the target name. A failure of that removal is returned wrapped in its own text, since the orphan would otherwise read as this entry's record: `rename saved the entry, but the orphaned decision record %s at the new slug's name could not be removed: %w: remove the file by hand, or the entry reads it as its own record`.
- For each slug of `loaded` that `after` lacks, it removes `decisionFileName(slug)`. An absent file is no error. This covers a remove, a prune and a merge that removes a slug it does not upsert again. Any other failure is logged at Warn through `internal/logger`, naming the entry and the file, and returns no error: an orphaned file is unread while its slug carries no flag, so `Board.RemoveTask`, `Board.Prune` and `Board.MergeTasks` gain no new error after their save.

The step returns an error only for the rename, as D4 prescribes.
That error does not stop the write it follows: `boardCriticalSection` holds the step's error, runs the render and the sync spawn as for any saved write, and returns the error last, so a saved rename is still rendered and pushed.

A merge that upserts a slug it removes keeps the slug in `after`, so its file stays, and card 1 keeps its flag.
A status-only write removes nothing, so `lym batten cancel`, which clears the status, and a landing that marks the entry done keep both the flag and the file until a prune.
Since the step runs after every post-mutation check, a rename refused by `checkNewSlugs` or any other check moves nothing.
Card 1's `noWrite` paths return before it.

A failed move reaches the caller as `Board.Rename`'s error after `board.json` is saved.
The new slug is then flagged with no file, which `DecisionRecord` answers as not found.
A failed removal of an orphan at the target name also reaches the caller as `Board.Rename`'s error after the save; the orphan then stays, and its error names it as the file to remove by hand.
`Board.Rename`'s doc comment says this: every refusal still comes before the save, and the record move follows it.
The `# Rename` section of `doc.go` says the record file moves with the entry, and the file-header line of `doc.go`'s `# Decision record` section says removals drop it.
Its `# Concurrency and sync` sequence names the record step between the save and the render.

`TestDecisionRecordFollowsEntry` (tier 0, external package) is a table.
Each row stores a record on its entry through `StoreDecisionRecord`, acts, and asserts the files and the flag:
- A rename moves the file to the new slug's name, and `DecisionRecord` on the new slug returns the bytes.
- A rename refused by a `ValidateSlug` seam that refuses the target leaves the file at the old name. This row builds its own `boardengine.Config`, with `newRenameBoard`'s fields and a `ValidateSlug` that refuses the target slug, since `newRenameBoard`'s config sets no `ValidateSlug` and so `checkNewSlugs` refuses nothing on it.
- A rename whose target `decision-<new>.md` already exists as a directory saves the rename, returns the move error naming the old file, leaves the old file in place, and still renders: the new slug's design file exists and the old slug's is gone.
- A rename of a flagged entry with no record file succeeds.
- A rename of a flagged entry with no record file onto a slug whose `decision-<new>.md` already exists removes that file, and `DecisionRecord` on the new slug answers found false.
- A remove deletes the file.
- A prune of a done entry deletes the file.
- A merge that removes the slug deletes its file.
- A merge that removes the slug and upserts it again keeps the file and the flag.
- `SetStatus(slug, nil)` keeps both.

**ImpactSummary:** One step joins boardengine's shared write scaffold, and it acts only when a write removes or renames an entry. Every Board write method passes through it, but the step only deletes or moves `decision-<slug>.md` files, so it touches no other file and no other package.
