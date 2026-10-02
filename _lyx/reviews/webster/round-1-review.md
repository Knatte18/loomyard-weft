---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: internal/boardengine/board.go:326
    summary: RetireLegacy drops legacy_done in the save that precedes the deletion, so a crash or a failed deletion leaves a surviving legacy file beside an empty legacy_done and the next load re-folds every legacy done mark, re-marking done any entry reopened after migration
  - id: F2
    severity: LOW
    location: internal/boardengine/task.go:1
    summary: doc comments on code the diff changed are stale - task.go file and Task doc still say tasks.json, MergeTasks doc cites the deleted writeOp, Board.Promote doc names idOrSlug for a slug parameter, and the ReadBoardType comments still say a type is corrected
  - id: F3
    severity: LOW
    location: internal/boardcli/text.go:22
    summary: RenderCompact pads columns by byte length, so a title holding a multi-byte character such as an em dash misaligns the status column the card says is aligned across lines
  - id: F4
    severity: LOW
    location: internal/boardengine/legacy.go:3
    summary: many new and changed Go comments are wrapped mid-sentence at a fixed column, or hold two sentences on one line, against CONSTRAINTS.md's Comment Line-Break Convention
  - id: F5
    severity: NIT
    location: internal/boardengine/board.go:357
    summary: PromoteNote reports note not found while notes are retired and every other path reports task not found
---

Review range: `product.parent` is `main` (`_lyx/shed/bolt-v2/status.json`), merge base `9ae5f9405`, commits `63e8b1337..4594739bc`, one per card.

The plan's verify block ran green on HEAD: `go build ./...`, `go test ./...`, the integration-tagged run over the six named package trees, and `go vet -tags smoke` over the same trees, each exiting 0.

### [MEDIUM] RetireLegacy's single save makes a crash or partial deletion re-fold every legacy done mark

**Location:** `internal/boardengine/board.go:326` (`RetireLegacy`), with the fold in `internal/boardengine/store.go:114` (`Load`).

**Issue:** The mutate sets `store.legacyDone = nil`, `boardCriticalSection` saves `board.json` without `legacy_done`, and only then does the after-save step delete the legacy files.
If the process dies between the save and the deletion, or the deletion fails part way (it removes `tasks.json`, then fails on `tasks.json.swaplock`, leaving `notes.json`), a legacy file survives beside a `board.json` whose `legacy_done` is empty.
The next `Load` runs `foldLegacyDone` with an empty `legacyDone`, so every `done` record in the surviving file is treated as a new mark: an entry that was `done` at migration and later reopened is set `done` again, and so is a new entry reusing a legacy-done slug.
The next write persists that.
The doc comment claims the opposite ("a crash in between leaves legacy files beside a store that already absorbed them"), and the plan's card 5 states the same property; the implementation does not deliver it.

**Fix:** Keep `legacy_done` through the first save, delete the legacy files and swap locks in the after-save step, and only then drop `legacy_done` and save `board.json` again.
A crash or failed deletion then leaves the legacy files beside an intact `legacy_done`, so a reload folds nothing twice and `retire-legacy` can simply be rerun.
A crash after the deletion but before the second save leaves an inert `legacy_done`, which `Load` never consults without a legacy file.
Add a regression test that forces the deletion to fail and asserts a reopened entry stays reopened.

### [LOW] Stale doc comments on code the diff changed

**Location:** `internal/boardengine/task.go:1`, `internal/boardengine/task.go:14`, `internal/boardengine/store.go:738`, `internal/boardengine/board.go:284`, `internal/battencli/wire.go:543-545`, `internal/battenshed/deps.go:140-142`.

**Issue:**
- `task.go`'s file comment ("the Task record stored in tasks.json") and the `Task` doc ("the canonical record stored in tasks.json") are wrong after card 4: the record is stored in `board.json`, and `tasks.json` is only a legacy migration source.
- `Store.MergeTasks`'s doc says "writeOp discards the in-memory mutation", but card 4 deleted `writeOp`; `boardCriticalSection` is what discards it.
- `Board.Promote`'s doc says "the entry identified by idOrSlug", but its parameter is `slug string`.
- `ReadBoardType`'s comments in `wire.go` and `deps.go` now say the seam returns the `recipe` field, but still justify the fresh read with "so a type corrected after prime was seeded"; `type` now names the entry kind, so this reads as the wrong field.

**Fix:** Name `board.json` in `task.go`, name `boardCriticalSection` in `MergeTasks`'s doc, name `slug` in `Board.Promote`'s doc, and say "a recipe corrected" in both `ReadBoardType` comments.

### [LOW] RenderCompact aligns by bytes, not characters

**Location:** `internal/boardcli/text.go:22-24` and `text.go:43-44` (`pad`).

**Issue:** Column widths come from `len(...)`, which counts bytes.
Board titles in this repo routinely carry an em dash (three bytes in UTF-8), so a title holding one is padded two columns short, and the `[<status>]` bracket after it no longer lines up with the others.
Card 7 asks for the columns to be aligned across lines.

**Fix:** Measure and pad with `utf8.RuneCountInString`, and add a `text_test.go` case with a multi-byte title followed by a status.

### [LOW] New comments break the Comment Line-Break Convention

**Location:** new and changed comments across the diff, among them:
- `internal/boardengine/legacy.go:3-5` (file comment), `legacy.go:137-138` (`foldLegacyDone`);
- `internal/boardengine/board.go:5-8` and `board.go:10-11` and `board.go:44-45` (package doc), `board.go:84-86` (`boardCriticalSection`), `board.go:174-175` (`MergeTasks`), `board.go:214-215` (`HealthCheck`), `board.go:324-325` (`RetireLegacy`);
- `internal/boardengine/store.go:3-6` (file comment), `store.go:65` (`NewStore`, two sentences on one line), `store.go:196-197` (`validateWrite`), `store.go:274-276` (`validateTier`), `store.go:579-580` (`Promote`), `store.go:612-613` (`Prune`), `store.go:642-643` (`Find`);
- `internal/boardengine/render.go:3-4` and `render.go:9-10` (file comment);
- `internal/boardcli/cli.go:3-9` (file comment), `cli.go`'s `storeVerbs` and `writeListing` docs;
- `internal/boardcli/text.go:1-2`, `text.go:16-17`;
- new test docs: `boardcli/verbs_test.go`'s file comment, `boardcli/cli_unit_test.go`'s `TestCLIAliasesHiddenWithShort`, `boardcli/notes_test.go`'s file comment and `TestCLINotesAliasReachesTopLevelEntries`, `boardcli/promotenote_test.go`'s `TestCLIPromoteNote` and `TestCLIPromoteNote_TierOneUnchanged`, `boardengine/render_test.go`'s file comment, `readmeFixture`, `TestRenderReadmeGolden` and `TestRenderDesignDocGoldens`, `boardengine/sync_integration_test.go`'s `TestSync_RetireLegacy_CommitsBoardJSONAndLegacyDeletions`, `loomcli/landingdeps_integration_test.go`'s `TestLandingDeps_MarkTaskDone_MigratesLegacyTasksJSON`.

**Issue:** CONSTRAINTS.md's Comment Line-Break Convention requires one sentence per line, broken further only at an independent-clause boundary, for new and changed Go comments.
These comments are wrapped near column 100 in the middle of a phrase (for example "the done-mark fold / that carries" and "whose entries carry a / tier and a type"), and some hold two sentences on one line.
Moved comments whose text the diff did not change, such as the verb comments inside `storeVerbs`, are legacy and out of scope.

**Fix:** Rejoin each comment to one sentence per line, breaking only at an independent-clause boundary.

### [NIT] PromoteNote's not-found message still says note

**Location:** `internal/boardengine/board.go:357`.

**Issue:** Notes are retired as a separate store, and `promote-note` is now an alias over the one store, yet its not-found error says `note not found` where `Store.Promote` and every other verb say `task not found`.

**Fix:** Report `task not found`, matching the store.

## Focus directives settled

1. Legacy compatibility window.
   A pre-upgrade binary writes only the legacy files under `board.lock`, and every new-binary write loads, and so folds, under the same lock, so an old-binary done mark is always folded before the next new-binary save and is never dropped.
   An unlocked read checks each file's existence before reading it (`readLegacyFile`, `fileExists`), so it creates no swap lock for an absent file, and `Load` never writes.
   `foldLegacyDone` skips a slug once it is in `legacy_done`, and duplicates within one call are skipped by the same check, so a fold neither double-applies across repeated loads nor drops a mark.
   An unlocked read that interleaves with a write can see a mix of states, as the pre-upgrade two-file read could; it persists nothing, so it is not a defect.
   `RetireLegacy`'s save-then-delete ordering does not hold on the crash and partial-deletion paths: F1.
   `noWrite` correctly skips the save, after-save step, render and sync, and a `fn` or save error returns before the after-save step runs.
2. Tier rule symmetry.
   `validateTier` checks both directions, and every write path reaches it: upsert, upsert-batch (each projected entry against the full projected snapshot), merge (the projection, then `UpsertTask` and `SetStatus`), set-deps, `Promote` and so `PromoteNote`, and `SetStatus` to any non-`done` value.
   `Promote` validates against a snapshot still holding the entry's old tier; direction 2 skips the entry's own slug, and a lower tier can only satisfy the dependents it already satisfied.
   A `done` entry's own deps stay checked, since the dependent's status never exempts an edge.
   Removal, prune and the fold only remove edges or mark entries done, so neither can introduce a violation.
   No path leaves a stored board that violates the rule.
   Not a defect.
3. Stale doc comments: F2 and F5.
4. Comment Line-Break Convention: F4.
5. Per-card mechanical checks.
   The verify block ran green on HEAD.
   Blast radius: the only production reader of the old recipe-meaning `Type` was `battencli/wire.go`, which the diff updates; no stencil reads `type`, `deferred` or the list order (the discussion stencil reads `get`'s `{"task": ...}` envelope, and landing's describe stencil reads `get`).
   `ListTasksBrief`, `Find` and `BriefTask` have no production caller outside `boardengine`/`boardcli`, the one-list `Render`/`RenderToDisk` likewise, and the deleted `*Note*` methods and `taskToUpsertFields` have no remaining reference.
   `ideengine`'s `Menu` uses only `HealthCheck`, which still passes on a legacy file alone.
   Create groups: nothing equivalent to `legacy.go`, `text.go`, `verbs_test.go` or the `ly:board` skill existed before the diff.
