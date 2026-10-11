# Card 7 — board-prune-retire-apply

**Edit:**
- `internal/boardengine#Board.Prune` -> `func (b *Board) Prune(apply bool) ([]string, error)`
- `internal/boardengine#Board.RetireLegacy` -> `func (b *Board) RetireLegacy(apply bool) ([]string, error)`
- `internal/boardengine#doneSection`
- `internal/boardengine_test#TestPruneRemovesDoneEntryAndDesignDocKeepsAbandoned`
- `internal/boardengine_test#TestRunLock`
- `internal/boardengine_test#TestPriorityRoundTrip`
- `internal/boardengine_test#TestRetireLegacyFoldsDoneMarkAndRemovesLegacyFiles`
- `internal/boardengine_test#TestRetireLegacyWithoutLegacyFilesRefusesAndWritesNothing`
- `internal/boardengine_test#TestRetireLegacyFailedDeletionKeepsLegacyDone`
- `internal/boardengine_test#TestRenderReadmeGolden`
- `internal/boardengine#stepSyncRetireLegacyCommitsBoardJSONAndLegacyDeletions`
- `internal/boardcli#Command`
- `internal/boardcli/doc.go`
- `internal/boardcli_test#stepPrune`
- `internal/boardcli_test#stepRetireLegacy`
- `internal/boardcli_test#TestHelpSchema_LeafCommands`
- `plugins/ly/skills/board/SKILL.md`

**Uses:**
- `internal/boardengine#Store.Prune`
- `internal/boardengine#noWrite`
- `internal/boardengine#Board.boardCriticalSection`
- `internal/boardengine#fileExists`

**Intent:** Decision D6 for `lyx board prune` and `lyx board retire-legacy`.
Both verbs become read-only dry runs without `--apply`, exit 0, print the fields an applied run prints filled with what they would change, plus `"applied": false`; with `--apply` each behaves as today and prints `"applied": true`.
Engine: `Board.Prune(apply)` runs `Store.Prune()` inside `boardCriticalSection` as today and, when `apply` is false, returns the removed slugs wrapped in `noWrite` so the store is neither saved nor rendered; the doc comment states both modes.
`Board.RetireLegacy(apply)` returns the legacy files present as their base names (`tasks.json`, `notes.json`), in that fixed order; it keeps refusing with "nothing to retire" when neither exists, in both modes, and with `apply` false it returns the list from inside the critical section through `noWrite` and runs no deletion, no `legacyDone` drop and no second save; with `apply` true it does what it does today and returns the same list.
README: `doneSection`'s text becomes "Finished, awaiting `lyx board prune --apply`.", and `TestRenderReadmeGolden`'s golden follows.
CLI: in `boardcli.Command`, give `prune` and `retire-legacy` a `--apply` bool flag read in each `RunE`; `prune` prints `{"removed": [...], "applied": <bool>}` and `retire-legacy` prints `{"legacy_files": [...], "applied": <bool>}` in both modes (an empty list prints as `[]`); rewrite each `Long` so its first sentence states the dry-run default and extend each example with the `--apply` form.
Record the dry-run default for both verbs in `internal/boardcli/doc.go`'s verb paragraph, and change the one `lyx board prune` line in `plugins/ly/skills/board/SKILL.md` to `lyx board prune --apply`.
Tests, all extending existing ones: retarget every caller of the two re-signed methods (`TestRunLock`'s prune row, `TestPriorityRoundTrip`'s prune row, `stepSyncRetireLegacyCommitsBoardJSONAndLegacyDeletions`, the four board tests listed) to pass `true` where today's behaviour is asserted; `TestPruneRemovesDoneEntryAndDesignDocKeepsAbandoned` first calls `Prune(false)` and asserts the done slug is reported while the entry, its design doc and `board.json` are untouched; `TestRetireLegacyFoldsDoneMarkAndRemovesLegacyFiles` first calls `RetireLegacy(false)` and asserts both file names are reported and every legacy file and swap lock still exists; `TestRetireLegacyWithoutLegacyFilesRefusesAndWritesNothing` asserts the refusal for both `apply` values.
In `stepPrune`, run bare `prune` first and assert `removed == [gone]`, `applied == false` and the entry still listed, then `prune --apply` with `applied == true`; in `stepRetireLegacy`, assert the refusal with and without `--apply`, then a bare run reports both files under `legacy_files` with `applied == false` and leaves them on disk, then `retire-legacy --apply` reports them with `applied == true` and removes them.
Add `--apply` to the `prune` row's `mustContain` in `TestHelpSchema_LeafCommands` and add a `retire-legacy` row with `mustContain` of `--apply`, `tasks.json` and `notes.json`.
Bound: `--apply` restores today's behaviour plus the `applied` field and `retire-legacy`'s `legacy_files` report; the dry run reads state only.

**ImpactSummary:** Two re-signed Board methods with callers in the CLI and six tests, one README string with its golden, and two verbs' flags, help and envelopes; no other board verb changes.
