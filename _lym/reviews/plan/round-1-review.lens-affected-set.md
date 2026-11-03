```yaml
lens: affected-set
outcome: reported
```

Task: affected-set lens — for each change the cards make, find what reads or asserts it and which cards cover that set.

**Per change: the affected set and the cards that cover it**

1. `Task.Discussed` field (card 1)
   - `taskKeys` in `internal/boardengine/task.go:70` reflects over `Task`'s JSON tags, so it picks up the new field without an edit.
   - `replayFields` is covered by card 4; the `get` envelope by card 5.
   - Covered in full.
2. `discussed` payload key and `UpsertPayloadKeys` (card 1)
   - `upsertAllowedKeys` is derived from `UpsertPayloadKeys`, so it needs no edit.
   - The upsert help in `boardcli#Command` is covered by card 1. `help_test.go:47-65` lists the keys it requires and does not require `discussed`.
   - The board skill lists no payload keys (no `on_land` hit in `SKILL.md`), so it needs no edit.
   - `boardengine/doc.go` is covered by card 1.
   - Covered.
3. `approve_seam` gains the value `discussion` and its error text changes (card 7)
   - `TestApproveSeam_FailsToBuild`'s unrecognized-value row is covered by card 7.
   - No test asserts the "must be %q" text (grep found none).
   - The `entries_bouncer.go` comments and the `entries_gate.go` header are covered by card 7.
   - No `doc.go` names `approve_seam` or `ApprovePlan` (grep found none).
   - Covered.
4. `Env.ApproveDiscussion` (card 7)
   - The Env builders are `loomcli/wiring.go:628`, `battencli/wire.go:917`, `envkit.go:295` and `loomcli/landingdeps.go:428`.
   - Card 7 covers loom `wire`, `FullEnv` and battencli's `intentionallyNil`.
   - The `landingdeps.go` Env builds only the Code-Bouncer gate through `CodePerchGate`, so it is unaffected.
   - Covered.
5. `approve_seam: discussion` on tapestry's Discussion-Bouncer (card 7)
   - `TestShippedRecipe_SeamsWiredOnPlanBouncerOnly`, `TestSequence_FullRunBlocksAtMergeIn` and the Plan-Bouncer comment are covered by card 7.
   - Covered.
6. `decision-&lt;slug&gt;.md` in the board directory (cards 1 and 4)
   - The render manifest never lists the file.
   - Board sync commits the whole tree.
   - `Claims`, `CommitRuns`, `Apply`, the `fabricengine.BoardDir` doc and the `boardengine` doc's `# Replay` section are covered by card 4.
   - Covered.
7. A rename moves the record after the save and can return an error after `board.json` is saved (card 2)
   - Card 2 updates the `Board.Rename` doc, the `rename.go` header and `doc.go`.
   - Not covered: `lym board rename` in `boardcli` and the rename section of `plugins/ly/skills/board/SKILL.md`. An error from that verb now can mean "saved, record not moved". See F1.
8. A write that removes an entry also removes its record file and can now return an error after the save (card 2)
   - The affected methods are `Board.RemoveTask` (`board.go:214`), `Board.Prune` (`board.go:393-397`, whose doc says only "the render that follows drops their design docs") and `Board.MergeTasks` (`board.go:221-226`).
   - Their callers include `boardcli` remove/prune/merge, `pairteardown/claim.go:159,172` and `fronttask/hub.go`.
   - No card covers them; card 2 updates only the rename docs. See F1.
9. `decision add` gains the `record_store_error` envelope key and the store pointer on its failed-commit refusals (card 8)
   - `add`'s `Long`, the `decision.go` header and the `loomcli` doc are covered by card 8.
   - `orch-role.md:79-142` names the verb but not the shape of its envelope, so nothing there is falsified.
   - Covered.
10. The `decision` group gains a second verb (card 9)
    - The group `Long` ("the only verb") and the file header are covered by card 9.
    - `plugins/ly/skills/cli/SKILL.md` holds no `decision add` text, so the index is generated rather than checked in.
    - Covered.
11. `afterFailedCommit` takes a new parameter (card 8)
    - Its one caller is `decisionVerb`, covered by card 8.
    - Not checked (out of calls): whether any test calls `afterFailedCommit` directly.
12. `LoadConfig` refuses some `design_prefix` values (card 3)
    - Callers: `ideengine/menu.go`, `adoptcli/fixloop.go`, configreg's `CheckLoad`, `boardcli`, every `OpenHub` caller and `FabricReplay.Apply`.
    - The fixture prefixes `proposal-`, `d-` and `design-` all pass, and the template default is `design-`.
    - Card 3 states this. Covered.
13. `lym board get --record` and the `discussed` field in its output (card 5)
    - `boardcli/doc.go`, the board skill's `get` section and `TestCLI` are covered by card 5.
    - Covered.

**Findings**

- **F1**
  - severity: MEDIUM
  - class: design
  - location: card 2 (`02-record-follows-entry.md`)
  - summary: Card 2 makes every write that removes or renames an entry able to fail after `board.json` is saved, but it re-documents only the rename path. These members of the affected set are missing:
    - the doc comments of `Board.RemoveTask`, `Board.Prune` and `Board.MergeTasks`;
    - the rename, remove, prune and merge help in `boardcli`;
    - the rename section of the board skill.

    Before this card, an error from these verbs meant nothing was saved. Callers that read it that way (`boardcli`, `pairteardown`, `fronttask`) now get a saved write reported as an error.
  - evidence:
    - `internal/boardengine/board.go:156-160` (`afterSave` errors return before the render today)
    - `internal/boardengine/board.go:214-219`, `:221-226` and `:393-397` (doc comments with no post-save error)
    - `internal/boardengine/rename.go:3` ("a refused rename writes nothing")
    - `pairteardown/claim.go:159,172` (callers, from the `OpenHub` grep)

**Tool calls**
1. Grep for tests that read `UpsertPayloadKeys`, `replayFields`, reflect over `Task`, or assert the `approve_seam` error text. My context did not show the test-side readers.
2. Read the reflective `taskKeys` in `task.go`, the payload-key help test, and searched for "only verb" assertions. These callers were not in my context.
3. Read `help_test.go` lines 40–80, listed the non-test `shedrecipe.Env{` builders, and listed docs naming `decision add`. These were needed to bound the Env and help sets.
4. Searched specs, stencils and `doc.go` files for `approve_seam` and for design-file listings. Needed to find doc text the cards could falsify.
5. Searched `doc.go` files for `ApprovePlan` and the board-file lists in the `fabricengine` and `boardengine` docs. Needed to confirm which docs describe the changed behaviour.
6. Checked the checked-in CLI skill index and `orch-role.md`'s `decision` references. Needed for card 9's verb-list set and card 8's envelope.
