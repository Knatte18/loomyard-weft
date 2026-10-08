# Webster recovery never dead-ends; computed Merriam base

Every Webster recovery instruction now names a lyx verb that works where it is read, inside or outside loom.
A recovery strand is told which uncommitted paths it starts from.
An operator decision taken mid-Webster reaches the plan, and a loom re-plan after a batch began no longer collides with the old run record.
A stale `batcher.yaml` fails at Loom-Preflight instead of mid-Webster, and the batchifier's Merriam start context is computed instead of fitted.

Closes #406
Closes #413
Closes #416
Closes #440
Closes #441

## What changes

### Reset targets (`websterengine`, `webstercli`)

- `lyx webster reset --to` gains `report-head --batch NN`, `last-batch-head` and `batch-start --batch NN`, each a commit the run recorded.
  `--batch` is required for the first and last of these and refused for the others.
- `report-head` is the one target allowed while the run lock is held, because Master calls it between a fork's report and `record-batch`.
  It is accepted only for the highest begun, non-terminal batch with a report on disk whose head descends from the batch's start, and only with no live recovery strand of that batch.
- `batch-start` refuses when a later batch recorded a start.
- `--to start` is now the start-over verb: after the reset it archives the run record, so a following plain `lyx webster run` starts a new run.
  When the start cannot be moved to (no recorded start, a missing start, starts with no common ancestor, or a start that is not an ancestor of HEAD) it archives without moving the branch and reports `moved: false` with the reason.
  The archive-only path judges the pending-findings guard against HEAD, not a recorded commit.
- Standalone mode performs the reset itself with a new `gitrepo` keep-reset instead of refusing with a git command.
  A dirty path git refuses to overwrite gets a restore step for a run-written path and commit-or-restore for any other.
- Every reset to start, in either mode, and every standalone reset lists the uncommitted paths it left in its envelope.
- No production Webster text names `git reset --keep` any more, and none names `lyx webster run --fresh` as a step after a reset; `cmd/lyx/websterretiredsteps_test.go` pins both.
  `run --fresh` stays as a CLI flag for a run outside loom.

### Recovery prompt on a dirty tree

`recover-batch` lists the worktree's uncommitted paths in the recovery prompt (`none` when clean), each classed by the run's own transcript-write evidence.
A path a session of this run wrote is kept as the starting point and checked against the card; any other path is left untouched and never staged.
This is the fix for #406.

### Decisions that reach a begun card

- `lyx webster rebaseline --card NN` accepts an edit to a card of a begun, non-terminal batch and records it in a per-batch `AmendedCards` list in `state.json`.
  The live attempt finishes on the old text; its report is then recorded failed with a `card_amended` finding, and the refusal carries `card_amended: true`.
  `recover-batch` renders every amended card into the recovery strand's failure digest and marks it rendered; a recovery that records done clears the list, and one that ends stuck, dead or failed leaves it.
  Master's ladder runs `recover-batch` on a `batch_failed` refusal carrying `card_amended: true`, even after a recovery.
- `rebaseline` accepts a `00-overview.md` change confined to its `## Card Index`, so a card added after the last begun batch lands.
  A second overview hash, over the file minus its Card Index, is stored in `state.json`; a state without it refuses every overview change as before.
  The refusals for a done batch's card and for any other overview change name the follow-up-card landing.
  `planparser` exposes the overview minus its Card Index.
- The sequence for a decision mid-Webster (`lyx loom decision add`, the card edit, `lyx webster rebaseline --card NN`) is documented in `websterengine/doc.go`; webster texts name no loom verb.
- Plan-Write's stale-plan rotation moves a present Webster run record into `archive-<stamp>/webster/` through `websterengine.ArchiveRunRecord`, and the plan commit includes it.
  The next Webster run starts a new run from the current HEAD, keeping the old run's commits, with no `plan_drifted` refusal.
  `PATTERN-plan-generation` is amended for this.

### Config failures

- A webster verb whose `batcher.yaml` or webster config fails to parse or validate refuses with `config_invalid: true` and changes no state, so a re-run of `record-batch` after the fix records the batch.
  An unreadable file refuses without `config_invalid` and names a re-run.
  `configengine` gains the `ErrInvalid` sentinel and `MarkInvalid` for this.
  Master's ladder ends the run `stuck` on a `config_invalid` refusal without calling another verb.
- `begin-batch`'s existing-report refusal names the one remedy for the recorded state: `record-batch` for a begun fork attempt, `recover-batch` for a begun recovery attempt or a terminal `stuck`/`failed`, end the run `stuck` for `dead`, begin the next batch for `done`.
- Loom-Preflight validates `batcher.yaml` through `batcher.Active` and fails with a `config` reason; for a retired key the way forward is `lyx config reconcile --apply`.
- `configreg.Module` gains a migration hook that `lyx config reconcile` runs, in the dry run and with `--apply`.
  Batcher's hook drops `alone_above`, `weights.startup_context` and `weights.master_base` from each profile, and adds `weights.orientation` at the template value only when the profile has none.
  The loader still refuses a retired key and never rewrites one.

### Computed Merriam base

`websterengine` computes Merriam's fixed part at partition time from the worktree's `CLAUDE.md` (and `CLAUDE.local.md`), the rendered Master stencil with the inlined PATTERN overview, and `00-overview.md`, plus a Go constant for the system prompt and tools.
`Batch` takes it beside `before`, and the start context is now computed base + `orientation` + (k−1) × `batch_growth`.
`orientation` replaces `master_base` in `Weights`, `template.yaml` (31400) and the calibrate fit in `tools/tokencount`.
A `state.json` breakdown carrying `master_base` still decodes, read-only.

## Design points for review

- `report-head` under a held run lock cannot see an in-session fork.
  The three checks (highest begun batch, no live recovery strand, report on disk) rule out every other writer; a Master that spawned a fork out of order is the residue.
- The archive-only reset accepts a branch rewritten under the run without moving it; the operator sees `moved: false` and the reason.
  After any reset the next run may start over uncommitted work: untracked files in either mode, tracked edits after a standalone or archive-only reset.
- Each amendment costs at most one extra recovery strand's cold start, because only a `rendered: false` entry forces the failed record.
- The Go constant behind the computed base is 20.6K minus the computed size of `CLAUDE.md` and the rendered Master stencil at this change's HEAD; the `template.yaml` value 31400 is the measured 52000 minus the measured 20.6K start.
  The base sizes text by lines, so a stencil with long lines prices low and only the fit absorbs that on average.
- Every new or reworded refusal has its row in `contracts/specs/refusal-spec.md` and a test that reaches it.

## To check by hand

- Run `lyx config reconcile` against a real pair `batcher.yaml` that still carries `startup_context` or `master_base`: the dry run lists each rewrite, `--apply` performs it, and a profile with its own `orientation` keeps it.
- Run `lyx webster reset --to report-head --batch NN` in a live loom run between a fork's report and `record-batch`, and confirm it refuses while a recovery strand of that batch is live.
- Rebaseline a card of an in-flight batch in a live run and confirm the failed record, the `card_amended` rung in Master's ladder and the recovery's digest.
- Do a loom `goto` back to Plan-Write after a batch began and confirm the run record lands under `archive-*/webster/` and the next run starts clean.
- Run the `tools/tokencount` calibrate fit over still-available measured runs; its `orientation` replaces 31400 when it differs.
  The fit was not run against real transcripts in this change.
- Deploy with `update-plugins.sh`, which refreshes the changed stencils (`webster-template-master`, `webster-prefix-recovery`).
