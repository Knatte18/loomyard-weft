# Decision record — webster-dead-ends

## Goal

Every Webster recovery instruction names a lyx verb that works where the instruction is read, inside or outside loom.
A recovery strand is told what uncommitted work it starts from and what to do with it.
An operator decision taken mid-Webster reaches the plan whether or not the card's batch has begun, and a loom re-plan after a batch began no longer collides with the old run record.
A record-batch that fails on config leaves a batch whose next step is named exactly, and a stale `batcher.yaml` fails at Loom-Preflight instead of mid-Webster.
The batchifier's Merriam start context is computed from what Merriam loads instead of a fitted `master_base`.

## Scope

In:

- `lyx webster reset` gains the targets `report-head`, `last-batch-head` and `batch-start`, and `--to start` becomes the one start-over verb (it archives the run record).
- Every way forward that names `git reset --keep` or `lyx webster run --fresh` is repointed at a `lyx webster reset` target, in `websterengine`, `webstercli` help text, `doc.go` and the refusal spec.
- Standalone mode performs the reset instead of refusing with a git command.
- The recovery prompt names the worktree's uncommitted paths with a keep-or-leave instruction per path.
- `lyx webster rebaseline --card NN` accepts an amendment to a card of the in-flight batch; the amended batch is re-run by recovery.
- Plan-Write's `archive-*/` rotation archives the Webster run record with the plan it retires.
- The begin-batch existing-report refusal names the one remedy for the recorded state; a config-load refusal from any webster verb carries `config_invalid: true` and Master's ladder stops on it.
- Loom-Preflight validates `batcher.yaml` through `batcher.Active`.
- `lyx config reconcile` migrates retired `batcher.yaml` profile keys (`startup_context`, `alone_above`, `master_base`).
- A computed Merriam base in `websterengine`, passed to `Batch`; `master_base` gives way to an `orientation` weight in `Weights`, `template.yaml` and the calibrate fit in `tools/tokencount`.

Out:

- A combined verb that edits the decision record and the plan in one call; the existing `lyx loom decision add` and `lyx webster rebaseline` stay separate verbs.
- Amending `00-overview.md` mid-run.
- Reopening a terminal `done` batch.
- Interrupting a live in-session fork.
- Validating config modules other than `batcher.yaml` at Loom-Preflight.
- Archiving Plan-Review or Webster-Review run directories on a Plan-Write rotation.
- The Webster producer's `Fresh: false` stays fixed.

## Decisions

### D1 — Reset targets

Decision: `lyx webster reset --to` accepts five targets, each a commit the run recorded:

- `start` and `pre-fix` as today.
- `report-head --batch NN`: the `head_sha` of batch NN's report on disk.
- `last-batch-head`: the last recorded batch head, picked by git ancestry exactly as accept-audit picks it (`runEvidenceBases`).
- `batch-start --batch NN`: batch NN's recorded `StartSHA`.

`--batch` is required for `report-head` and `batch-start` and refused for the others.
Every target keeps `PlanReset`'s existing guards (merge in progress, foreign branch, missing commit, target not an ancestor of HEAD, foreign dirty paths) and performs the reset through fabric's pair-checkout reset.

Bound, per new target:

- `report-head` reads a fork-written SHA, so it is accepted only when batch NN is begun and not terminal, its report is on disk, and the report head descends from (or equals) batch NN's recorded `StartSHA`; it can discard only commits above that batch's report head.
- `report-head` is the one target allowed while the run lock is held, because its caller is Master inside the run, between the fork's report and record-batch, when no fork of the run is in flight; every other target keeps the run-lock refusal.
- `batch-start` refuses when any later batch recorded a start, so it never discards another batch's commits; its way forward then names `--to start`.
- `last-batch-head` and `batch-start` refuse when the state recorded no such commit, naming `--to start`.

Rationale: three recovery sites and standalone mode still name `git reset --keep`; each moves back to a commit the run recorded, so each gets a named target instead of a raw SHA.
Master hits the report-head mismatch inside its run, where today it runs `git reset --keep` itself; a lyx verb that refused under the run lock would turn that into a dead end.

### D2 — Sites repointed

Decision: the way forwards change as follows.

- `reconcileHead`'s refusal for record-batch and recover-batch names `lyx webster reset --to report-head --batch NN`, then the re-run, then the parent merge-in redo where it applies.
- Its refusal for accept-audit names `lyx webster reset --to last-batch-head`.
- accept-audit `--batch`'s "HEAD is not batch NN's start" refusal names `lyx webster reset --to batch-start --batch NN` when the start is an ancestor of HEAD, and `lyx webster reset --to start` when it is not.
- recover-batch's needs-fresh way forward names `lyx webster reset --to batch-start --batch NN` in place of the parenthetical `git reset --keep`.
- `resetKeepStep` is deleted.
- accept-audit's help text names the lyx target.

Rationale: after this task no Webster text names `git reset --keep`, so an agent or operator never needs a raw git reset.

### D3 — `reset --to start` is the start-over verb

Decision: `lyx webster reset --to start` ends by archiving the run record, through the same archive `lyx webster run --fresh` uses (`ArchiveRunRecord`), after the branch reset and state save succeed.
When no batch recorded a start, or the recorded start is not an ancestor of HEAD, it archives the record without moving the branch and reports `moved: false` with the reason in its envelope.
The next `lyx webster run`, plain or from the Webster producer with `Fresh: false`, then starts a new run.
Every way forward that today reads "`lyx webster reset --to start`, then `lyx webster run --fresh`" becomes "`lyx webster reset --to start`, then `lyx webster run`"; a refusal surfaced through the Webster producer keeps ending in the row's re-entry step.

Bound: the archive-only path runs no git and moves no commit; it moves the run record into an archive directory, never deletes it.
`run --fresh` stays as a CLI flag for a run outside loom.

Rationale: every reset to start is followed by a fresh run, and the producer can never run fresh, so the pair "reset, then `run --fresh`" had no path through loom.
Folding the archive into the reset gives one verb an orch or operator runs, then re-steps.

### D4 — Standalone reset

Decision: in standalone mode, `lyx webster reset` performs the planned reset with `git reset --keep <sha>` through `internal/gitrepo` (a keep-reset beside the existing hard reset, on the `gitexec` side of the client boundary), and archives the record for `--to start` as D3 does.
It touches no remote.
When `--keep` refuses on a dirty path, the verb surfaces git's refusal with the way forward to commit or restore the path, then re-run.

Bound: standalone has no fabric gate, so the keep form is the guard: it refuses rather than discards uncommitted changes, unlike the pair path, which discards changes to the run's own paths.

Rationale: standalone mode was the last place a webster refusal named a git command.

### D5 — Recovery prompt names a dirty tree

Decision: when recover-batch spawns a recovery strand, it lists the worktree's uncommitted paths (tracked changes and untracked files) and renders them in the recovery prompt through a new optional marker in `webster-prefix-recovery.md`, `none` when clean.
Each path is classed by the run's own write evidence, the transcript writes `ownTrackedPaths` already reads:

- A path a session of this run wrote: the earlier attempt's unfinished work; keep it as the starting point, check it against the card before building on it, and apply the suspect rule to any path the failure digest names.
- A path no session of this run wrote: not this run's; leave it untouched and do not stage or commit it.

The stencil's "You continue from the committed tree" sentence is rewritten so it no longer contradicts a dirty tree.

Rationale: the issue's agent had to judge on its own whether a large uncommitted body of work was to be kept; the run's write evidence answers that per path without a guess.

### D6 — An amendment reaches the in-flight batch

Decision: `lyx webster rebaseline --card NN` accepts an edit to a card of a begun, non-terminal batch.
It restamps the card's hash as it does for a terminal failed batch, and records the card in a new per-batch `AmendedCards` list in `state.json`.
The live attempt (fork or recovery strand) keeps running on the old text.
When that attempt's report is recorded (record-batch, or recovery's terminal persist), the batch runs every normal check and is then recorded failed with an added `card_amended` finding naming each amended card, whatever the report's status.
recover-batch treats `card_amended` as a recoverable finding (never uncheckable, never `needs_fresh`), renders it in the failure digest with an instruction to re-read the amended card and bring the committed work in line, and clears the batch's `AmendedCards` once the recovery strand is spawned.
Master's ladder runs `recover-batch` for a `card_amended` failure even when the failed attempt was itself a recovery.

Bound: only the in-flight batch, only cards named with `--card`, never a change to a batch's card-ID set, never `00-overview.md`.
Each amendment costs at most one extra recovery, because the mark clears when a recovery renders it.

Rationale: the operator's decision against a running card must land in the card itself so the plan and the implementation agree; recording the batch failed reuses the existing failed-batch recovery path instead of a new one.

### D7 — A decision against a done card or the overview

Decision: rebaseline keeps refusing an edit to a card of a terminal `done` batch and to `00-overview.md`, and the refusal's way forward names the landing that works: restore the card, then add a follow-up card after the last begun batch that carries the decision, then `lyx webster rebaseline --card NN` naming it.
For `00-overview.md`, the way forward names carrying the decision in the cards it affects.

Bound: a done batch's commits are never reopened by a plan edit; the follow-up card is an ordinary card in the tail rebaseline already accepts.

Rationale: reopening a done batch would rework code later cards may already build on; the follow-up card lands the decision without that risk and without new machinery.

### D8 — Decision record and plan together

Decision: the sequence for an operator decision mid-Webster is `lyx loom decision add` for the record, the card edit, then `lyx webster rebaseline --card NN`.
It is documented in webster's `doc.go` rebaseline section and named in the in-flight refusal's way forward only as the webster half; webster texts name no loom verb.

Rationale: both verbs exist and commit their own files; webster runs standalone too, so it stays loom-agnostic.

### D9 — A loom re-plan archives the Webster run record

Decision: when Plan-Write's stale-plan rotation moves the live plan into `archive-<stamp>/`, it also moves a present Webster run record into `archive-<stamp>/webster/` through `websterengine.ArchiveRunRecord`, told the destination, as PR-Rework does for a round.
The next Webster run starts a new run from the current HEAD, keeping the commits the old run made, since the new plan was written against that tree.
`PATTERN-plan-generation` is amended in the same commit: Webster's run record also moves with Plan-Write's `archive-*/` rotation.

Bound: the move runs only inside the rotation, only when a run record exists, and moves files without deleting or resetting anything.

Rationale: a `goto` back to Plan-Write retires the plan generation, and the run record is generation-scoped; leaving it behind made record-batch refuse `plan_drifted` and rebaseline refuse the begun card, a loop with no exit that kept the run's progress.

### D10 — record-batch config failure and begin-batch's remedy

Decision: a webster verb whose config fails to load (the wiring's `batcher.yaml` and webster config loads) refuses with `config_invalid: true` in its envelope and changes no state; a re-run after the fix proceeds normally, so record-batch then records the batch.
The begin-batch existing-report refusal names one remedy, picked from the recorded state:

- Begun, not terminal, fork attempt: `lyx webster record-batch NN`, after fixing whatever its last refusal named.
- Begun, not terminal, recovery attempt: `lyx webster recover-batch NN`.
- Terminal `stuck`, `dead` or `failed`: `lyx webster recover-batch NN`.
- Terminal `done`: the batch is finished; begin the next batch.

Master's ladder gains a rung: a refusal carrying `config_invalid: true` from any verb ends the run with `outcome: stuck`, quoting the refusal, without calling another verb.

Rationale: the issue's batch was already recoverable by re-running record-batch, but begin-batch listed several remedies and Master picked the wrong one; config is not Master's to fix.

### D11 — Loom-Preflight validates batcher.yaml

Decision: Loom-Preflight calls `batcher.Active` on the worktree's config, the same load the Batchifier row and webster's wiring make, and fails with a `config` reason naming `batcher.yaml` and the way forward.
When the failure is a retired key, the way forward is `lyx config reconcile --apply`, then re-step; otherwise it is the Batchifier row's existing fix-and-re-step text.

Rationale: the drift surfaced five rows into the run; Loom-Preflight is the first row that is loom's own, and `batcher` is a loom concern, not the general Preflight's.

### D12 — Reconcile migrates retired batcher keys

Decision: `configreg.Module` gains a migration hook that `lyx config reconcile` runs; batcher's hook rewrites, inside each `profiles` entry:

- drops `startup_context` and `alone_above`;
- replaces `weights.master_base` with `weights.orientation` at the template's value.

Reconcile reports each rewrite as a `migrated` entry, in the dry run and with `--apply`.
`batcher.Active` keeps refusing a retired key, and its message names `lyx config reconcile --apply`.
webster's `batcherLoadError` names the same step for a retired key.

Bound: the hook rewrites only the keys a module declares retired; every other open-map entry is still carried whole.

Rationale: `profiles` is an open map, which reconcile carries whole, so a retired key inside a profile survived every reconcile; retiring `master_base` in this task would otherwise break every seeded `batcher.yaml` on the first run after deploy.

### D13 — Computed Merriam base

Decision: a function in `websterengine` computes Merriam's fixed part at partition time: the worktree's `CLAUDE.md` (and `CLAUDE.local.md` when present), the rendered Master stencil with the inlined PATTERN overview, and `00-overview.md`, the one plan file Merriam reads up front; each is sized with the batcher's own size model (lines × `context_per_line`), plus a Go constant for the system prompt and tools, set from the measured 20.6K start.
`Batch` takes the computed base beside `before`, and `startContext` becomes `computedBase + orientation + (k−1) × batch_growth`.
`orientation` replaces `master_base` in `Weights`, `template.yaml` and the calibrate fit in `tools/tokencount`, which fits `orientation` against measured starts minus each run's computed base.
`template.yaml`'s `orientation` value comes from that fit over the existing measured runs.
A breakdown recorded in an older `state.json` with `master_base` still decodes, read-only, as `startup_context` does today.
The identity batchifier ignores the base, as it ignores `before`.

Rationale: the base is fixed by what Merriam loads, so a bigger plan or another repo lands far from a fitted 52K; computing the loaded part leaves only the orientation work to fit.

## Constraints

- `PATTERN-refusal-way-forward`: every new or reworded refusal (reset targets, rebaseline's in-flight and done-card refusals, begin-batch's remedy, `config_invalid`, Loom-Preflight's batcher check) names its way forward, has its row in `contracts/specs/refusal-spec.md` in the same commit, and is reached by a test.
- `PATTERN-no-denied-recovery`: no way forward names `git reset --hard`, `git push --force` or `rm -rf`; after this task none names `git reset --keep` either.
- `PATTERN-fabric-git`, `PATTERN-gitrepo-client-boundary`, `PATTERN-gitexec-checked-call`: the pair reset stays fabric's `ResetPairCode`; the standalone keep-reset is a `gitrepo` method on the `gitexec` side; webster runs no raw git.
- `PATTERN-fabric-destruction-chokepoint`: the pair reset already goes through `destroy.go`; the new targets add no other destruction path.
- `PATTERN-plan-generation`: amended in the same commit for D9.
- `PATTERN-told-geometry`: `loomshed` gets the Webster archive destination and the batcher config dir as told paths; `websterengine`'s base function is handed its paths.
- `PATTERN-config-strictness`: `batcher.Active` keeps its load mode; the migration lives in reconcile, never in the loader.
- `PATTERN-batcher-registry`: the partition is still recorded at first init and replaced only by a rebaseline; the computed base enters at partition time only.
- `PATTERN-stencil-ownership`, `PATTERN-parent-directive`: the recovery prefix's new marker is filled at render time from the told stencil dir; no stencil asks an agent to ask the operator.
- `PATTERN-sole-parsers`: plan files are read through `planparser`.
- `PATTERN-friction-capture`: the new refusals go through the existing `noteRefusals` wrapper.
- `PATTERN-leaf-packages`, `PATTERN-shed-producer-seam`: no new import into a closed set.
- `PATTERN-test-economy`, `PATTERN-test-speed`, `PATTERN-test-isolation`, `PATTERN-hubforge-fixtures`: tests at module public surfaces, lowest tier that holds the behavior, extend existing table tests.
- `PATTERN-comment-line-breaks` and the docs rule: `websterengine/doc.go`, `batcher/doc.go`, `loomshed` doc, the reset and accept-audit help text and the Master template land in the same commit.

## Auto-mode assumptions

- No operator answered; every decision above is an auto-pick, logged in the support log's question ledger.
- The issue #440 run passed the Batchifier row, which already calls `batcher.Active`, so its config most likely changed under the run (a deploy between rows); Loom-Preflight catches only a stale config at run start, and D10's `config_invalid` stop and D12's migration cover a mid-run deploy.
- In-session forks cannot be interrupted, so D6 lets the old attempt finish and re-runs the batch by recovery.
- Merriam reads only `00-overview.md` up front (its stencil says so), so the other plan files stay out of the computed base.
- The system-prompt-and-tools constant is the measured 20.6K start; drift in it is absorbed by refitting `orientation`.

## Open risks

- D6's extra recovery costs a recovery strand's full cold start per amendment.
- D3's archive-only path accepts a branch rewritten under the run without moving it; the operator sees `moved: false` and the reason, but nothing else stops them.
- `report-head` under the run lock trusts that no fork is in flight between a fork's report and record-batch; a Master that forks out of order could have a fork writing during the reset.
- The computed base sizes text by lines, so a stencil with long lines prices low; the fit absorbs the bias only on average.
- D12's `orientation` value depends on rerunning the calibrate fit over existing measurements; if those transcripts are gone, the plan has to derive the value from `52000` minus the computed base of the measured runs' plans.

## Acceptance criteria

- `lyx webster reset --to report-head --batch NN` moves HEAD to the report's `head_sha` while the run lock is held, and refuses for a terminal batch, a missing report, and a report head that does not descend from the batch's start.
- `lyx webster reset --to last-batch-head` and `--to batch-start --batch NN` move HEAD to their recorded commits; `batch-start` refuses when a later batch recorded a start; both refuse under a held run lock.
- `--batch` is refused with `start`, `pre-fix` and `last-batch-head`, and required with `report-head` and `batch-start`.
- `lyx webster reset --to start` archives the run record so a following plain `lyx webster run` starts a new run; with no recorded start or a non-ancestor start it archives without moving and reports `moved: false`.
- No production Webster text contains `git reset --keep` or names `lyx webster run --fresh` as a step after a reset; a test pins the absence.
- Standalone `lyx webster reset` moves HEAD with the keep form and surfaces git's refusal on a dirty path it would overwrite.
- A recovery prompt rendered over a dirty worktree lists each uncommitted path under the keep or the leave instruction by the run's write evidence, and renders `none` on a clean tree.
- `lyx webster rebaseline --card NN` on an in-flight batch's card is accepted; the batch's next recorded report yields a failed batch with a `card_amended` finding even when the report says done; recover-batch renders the amendment in the failure digest, does not answer `needs_fresh`, and clears the mark.
- rebaseline on a done batch's card and on `00-overview.md` refuses, naming the follow-up card or the card-level landing.
- A loom `goto` to Plan-Write after a batch began leaves the old run record under the plan's `archive-*/webster/`, and the next Webster run starts a new run without a `plan_drifted` refusal.
- A webster verb whose `batcher.yaml` fails to load refuses with `config_invalid: true` and leaves `state.json` and the report unchanged; record-batch re-run after the fix records the batch.
- begin-batch's existing-report refusal names exactly one remedy for each recorded state in D10.
- Loom-Preflight fails on a `batcher.yaml` carrying `startup_context` or `master_base`, naming `lyx config reconcile --apply`.
- `lyx config reconcile` reports and, with `--apply`, performs the batcher migration, leaving every other profile entry intact.
- The cost batchifier's start context equals computed base plus `orientation` plus `(k−1) × batch_growth`; the computed base grows with `00-overview.md` and `CLAUDE.md`; a `state.json` breakdown carrying `master_base` still decodes.
- The calibrate fit in `tools/tokencount` reports `orientation` against measured starts minus the computed base.

## Notes for the plan writer

- Reset: `internal/websterengine/reset.go` (`PlanReset`, `ResetTarget`), `internal/webstercli/reset.go`; the sites are `gitwrap.go` (`reconcileHead`, `parentMergeRefusal`, `resetKeepStep`), `auditledger.go` (`acceptAuditHeadRefusal`, `AcceptBatchFabricReference`), `recoverbatch.go` (`RecoverSpawnOrAttach`'s needs-fresh text), `findingtext.go` and `planbaseline.go` (`freshRestartSteps`, `stepRunFresh`).
- The archive is `ArchiveRunRecord` in `websterengine/archive.go`; loom already passes it a told destination in `internal/loomcli/wiring.go` for PR-Rework.
- Plan-Write's rotation is `stalePlanRotation`/`rotateStalePlanDir` in `internal/loomshed/planwrite.go`.
- Rebaseline's begun-batch bound is in `rebaseline.go`; the card-hash check that turns an edit into `plan_drifted` is `batchCardEditError` in `fingerprint.go`.
- The recovery prompt is composed in `render.go` from `contracts/stencils/webster/webster-prefix-recovery.md`; Master's ladder is `webster-template-master.md`.
- The begin-batch existing-report refusal is in `beginbatch.go`; the config load refusals are `batcherLoadError` and its sibling in `internal/webstercli/wiring.go`.
- Loom-Preflight is `internal/loomshed/loompreflight.go`; the Batchifier row's `batcher.Active` gate is `internal/loomshed/batchifier.go`.
- The retired-key refusal is in `internal/batcher/estimate.go`; reconcile is configsync behind `configreg.Module`.
- `Batch` is called once, from `websterengine/partition.go`.
