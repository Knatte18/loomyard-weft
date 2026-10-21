# Plan round 1 — fixer report

Every finding's premise held against the code and the decision record.
`lyx loom validate-plan` returns ok after the fixes and after the self-check edits.

## Changes

- **F1** (fixed as proposed, plus one site).
  - Card 7 adds `internal/configreg#TestModules_EveryTemplateKeyClassified` and `internal/darnengine/doc.go` to its Edit list.
    Its Tests add `review_amend_max_bounces` to darn's `wantLocalKeys` after `driver_kind`.
    Its Intent extends darnengine's doc.go key list and its budget-keys sentence to three keys, and `loomengine/config.go`'s file-header key list.
  - The one site beyond the proposal: darnengine's doc.go `LoadConfig` paragraph also lists the budget refusals (`internal/darnengine/doc.go:12`), so it names the new below-0 refusal too.
  - Card 5 adds `internal/loomcli#TestStatusCmd_EnvelopeKeySet` to its Edit list, and its Tests add `pause_reason` to that test's `wantKeys`.
  - Card 20's overview sentence names `review_amend_max_bounces` in the darn and loom config key descriptions.
- **F2** (fixed as proposed).
  Card 11's Writes paragraph gates every settle-record and generation-record write on a told `Subject` seam answering true; otherwise nothing is written, so re-entry takes case 4.
  `TestBouncer_Reentry` gains a nil-`Subject` row that asserts no `generation-record.json` exists after a seed and a judge.
- **F3** (fixed as proposed, then refined by the self-check).
  In card 14, `recreateReviewRunDirs` calls `KeepReviewRecords` on every Call that reaches it.
  It does so for each `ReviewRunSubdirs` entry present under `LatestArchivedReviewsDir(reworkDir)`, whether or not this Call archived.
  The `TestPRRework_FirstRoundMovesGenerationZero` failure row now continues: the re-step on the in-flight round carries the records.
- **F4** (fixed as proposed).
  Card 10 gains the Bound sentence after `Unchanged`, carrying D5's statement of what the patch-id admission lets through and what bounds it.
- **F5** (fixed differently in placement).
  The review proposed adding `JudgeAt` in card 12's wording.
  I defined the field in card 10's `generationRecord` and had card 11's `judgeCall` write it, because those cards own the record and its writes.
  Defining it in card 12 would have card 12 change a type card 10 creates and a write card 11 owns.
  Card 12's `startGeneration` compares a reject against `SettledAt` in case 2 and `JudgeAt` in case 3.
  `TestBouncer_Amendment` gains the row: case 3 with a reject older than `JudgeAt` names no reject file.
- **F6** (fixed as proposed, wording refined by the self-check).
  Card 16's refusal ends "and a changed subject then starts a new generation".
  `TestSettledSegmentGuard` gains a Merge-Review row that names `Merge-Bouncer` and `Merge-In`.
- **F7** (fixed as proposed).
  Card 17 has `battenCLI.wire` build batten's clause from its resolved `slug` argument, the run-id, rendering `run "lyx batten run <run-id>"` with the real value.
  It notes beside the loom clause why loom's clause needs no run-id.
- **F8** (fixed as proposed).
  Card 16's way forward adds `then "lyx loom resume" in the task worktree runs it` after the goto onto the Bouncer.
- **F9** (fixed as proposed).
  Card 17's Create signature carries the constant's real text.

## Self-check

- fix-claims, LOW: card 14's "on every Call" is false for Calls that return before `recreateReviewRunDirs` (`prrework.go:130-150`).
  Fixed as proposed: "on every Call that reaches it".
- fix-proposal, MEDIUM: the F3 carry could overwrite fresh records.
  Case: an in-flight round, then a goto past it and a Bouncer settle, then a rekeyed rejection.
  Fixed with my own change.
  The fork proposed copy-if-absent keyed on the settle record; I rejected that, because a mid-generation directory holds a generation record and no settle record, so it would still be overwritten.
  Instead, card 10's `CarryRecords` copies nothing into a target that already holds a generation record, and copies that record last, so a part-way carry is redone.
  Card 14 states the rule, and card 10 gains `TestCarryRecords`, because `archiveRunDir`'s empty target never reaches the skip.
- fix-proposal, LOW: card 16's "rewrites the subject" is wrong for a no-op Merge-In.
  Fixed as proposed: "re-runs the writer".
- fix-consistency: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
