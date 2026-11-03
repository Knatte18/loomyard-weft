# Card 4 — replay-carries-record

**Create:**
- `plan:internal/boardengine#boardRecordSlug` -> `func boardRecordSlug(path string) (string, bool)`

**Edit:**
- `internal/boardengine#FabricReplay.Claims`
- `internal/boardengine#FabricReplay.CommitRuns`
- `internal/boardengine#FabricReplay.Apply`
- `internal/boardengine#replayFields`
- `internal/boardengine#replayRun.applyDesignFile` -> `func (r *replayRun) applyDesignFile(boardDir, label string, file fabricengine.ReplayFile) (bool, error)`
- `internal/boardengine#TestFabricReplay`
- `internal/boardengine#TestFabricReplay_Ledger`
- `internal/boardengine/doc.go`
- `internal/fabricengine#BoardDir`

**Uses:**
- `internal/fabricengine/doc.go`
- `plan:internal/boardengine#decisionFilePrefix`
- `plan:internal/boardengine#decisionFileSuffix`
- `internal/boardengine#boardDesignSlug`
- `internal/boardengine#hubWithBoardConfig`
- `internal/boardengine#boardJSON`
- `internal/boardengine#replayEntry`

**Intent:**
Make the board replay carry the record file and the flag (D5).

`boardRecordSlug(path)` mirrors `boardDesignSlug` over `decisionFilePrefix` and `decisionFileSuffix`: a `decision-<slug>.md` at the Bolt root, with a non-empty slug and no `/`.

The replay changes in four places:
- `FabricReplay.Claims` claims such a path.
- `FabricReplay.CommitRuns` adds its slug, as it does a design file's.
- `replayFields` gains a `discussed` field, `Get` and `Set` over `Task.Discussed`, so the flag merges by entry like status.
- `FabricReplay.Apply` collects record files beside the design files.

`Apply` applies the record files after the entry changes, so origin's store already reflects them.
- A record file whose commit side writes bytes (`After` non-nil) for a slug that origin's store does not hold is reported lost, through `lost(path, "decision record", "origin does not hold the entry", "")`, and nothing is written.
- Every other record file, a removal included, goes through the design-file rule. Applied when origin's file is still as the merge base or the commit's parent had it; otherwise origin's file is kept and the local one reported lost. So a replayed removal or rename never leaves the old file on origin.
- An applied record file counts as an applied file, so the re-render runs.

`applyDesignFile` takes the label it reports under as a parameter, `"design file"` or `"decision record"`.
Its doc comment says it serves both kinds of file.

`doc.go`'s `# Replay` section names the `decision-<slug>.md` files among the claimed files, the `discussed` field among the merged fields, and the entry check on a written record file.
`fabricengine.BoardDir`'s doc comment, which lists what the board directory holds as board.json, its overview and design files, adds the decision records.

Test rows, tier 0 as the tables already are.

New `TestFabricReplay` rows, each with record files in `designs`:
- A commit that adds a record file lands it when origin lacks the file.
- A commit that changes a record file lands the change when origin's file is as the parent had it.
- A commit that removes a record file removes it when origin's file is untouched.
- A change on both sides keeps origin's file and reports it lost.
- A written record file whose entry origin lacks is reported lost and leaves origin without the file.
- A commit removing an entry and its record file leaves no record file on origin.
- A rename commit, the old entry and its file removed and the new ones added, leaves no old record file on origin.
- A commit setting `discussed` on an entry lands the flag on origin's entry.

New `TestFabricReplay_Ledger` rows:
- `Claims` accepts `decision-x.md` and refuses `sub/decision-x.md` and `decision-.md`.
- `CommitRuns` names the slug of a record file the commit touches.

**ImpactSummary:** Inside boardengine's replay only: one more claimed file kind, one more merged field and one more file-apply branch. fabricengine reaches the replayer only through its registry interface, so no caller outside the package changes.
