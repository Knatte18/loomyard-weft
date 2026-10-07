# Round 2 fixer report

Every finding in `round-2-review.md` was fixed in the plan files; nothing was deferred.
`lyx loom validate-plan` returns `ok: true` after the changes, and `approved:` in `00-overview.md` is untouched (`false`).

## Changes

### F1 (BLOCKING) — Card 6 missed a test asserting the reworded refusal

`06-neutral-env-and-refusal.md`:
- Added `internal/fabriccli/cleanhub_integration_test.go` to the Edit list; its `"weft sibling of a pair"` assertion moves to the new wording.
- Added `tools/sandbox/SANDBOX-FABRIC-SUITE.md` as a Prosa target, so its instruction to confirm a refusal "naming the weft sibling" names the records sibling.
- Rewrote the sentence listing the wording assertions: `battencli`'s `lifecycle_integration_test.go` and `fabriccli`'s `cleanhub_integration_test.go` assert it, and `fabric_test.go` asserts only `ErrNotAWarpWorktree`, so it stays a target for `FABRIC_SKIP_*` alone.

### F2 (BLOCKING) — Card 3 left `PushPushesBothSides` pinning the old prime behaviour

`03-prime-records-only-push.md` Coverage: `PushPushesBothSides` becomes the task-pair case (adds a pair, pushes from the pair's code worktree, asserts both bares advance and no `code_push_skipped`), and `PushReportsADivergedWarpSideAndStillPushesWeft` stays the one prime case, so the two steps no longer cover one behaviour twice.

### F3 (MEDIUM) — statements D1 and D2 falsify, in no card

- `02-rollback-created-warp-branch.md`: added `internal/fabricengine/doc.go`, `internal/battencli/wire.go`, `internal/battencli/lifecycle_integration_test.go` and `internal/landingshed/publish.go` to the Edit list and `docs/overview.md` to Prosa.
  The Intent names each rewording: the `doc.go` rollback sentence, `createRefusal`'s comment and message, the `lifecycle_integration_test.go` doc comment, the `publish.go` Stuck rationale and the `overview.md` sentence.
  The ImpactSummary now lists those changes.
- `01-add-push-retry.md`: added `internal/fabricengine/doc.go` to the Edit list with one sentence on the bounded retry, and updated the ImpactSummary.

### F4 (MEDIUM) — quarry-indexed renames typed Edit

- `05-fabricengine-neutral-exports.md`: `WeftBranchName`, `WeftRepoRoot`, `CommitWeftPaths` and `WarpLyxLink` are now a `Rename` group of `old glyph -> plan:` handle pairs, so their callers leave the Edit list.
  `WeftWorktree`, `WeftWorktreePath` and `Fabric.ResetPairWarp` stay `Edit`: the gate reports `containment-file-overlap` for them, because `fabric.go`, `weftwiring.go` and `destroy.go` are each a whole-file target of two other cards (confirmed by running `lyx loom validate-plan` with all seven as `Rename`).
  The Edit list was recomputed from the tree: every file that calls one of the three, plus every file that spells any of the seven in a comment or a doc comment, plus the two test files Cards 1 and 2 create.
- `04-hubforge-neutral-names.md`: stays `Edit` for the three `Hub` methods, now with the reason stated: `hub.go` also holds call sites Card 5 edits, and a member glyph beside another card's whole-file target is blocked (the validator confirmed this too).
  The two fields are not indexed by quarry.
- `00-overview.md` Shared Decisions: the blanket sentence is replaced by two bullets giving the real reasons per card.

### F5 (MEDIUM) and F6 (LOW) — Card 3's test budget and failure rows

`03-prime-records-only-push.md`: the fabricengine cases are stated to be `t.Run` rows of `TestCommit_InvokesPushRecorder` and `TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide`, adding no top-level function, so the budget row needs no raise.
The prime-name failure rows now name their mechanism: a `Location` whose `AnchorRel` names an absent subdirectory for `PushPairAnchored`, and a recorded anchor naming one for `Commit`; `Commit` resolves the prime directly after its layout resolution so that error is the one the row sees.

### F7 (NIT) — Card 5 over-listed a Card 7 file

`internal/ideengine/spawn_hub_integration_test.go` is out of Card 5's Edit list; it spells only the result-struct fields Card 7 renames.

### F8 (NIT) — Card 10 missed one comment

`10-delete-vocab-exemption.md` now names `bareVocabularyTokenIn`'s doc comment among the comments it rewrites.

## Deferred

Nothing.
