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
- `lyx webster rebaseline --card NN` accepts an amendment to a card of the in-flight batch; the amended batch is re-run by recovery, and Master's ladder runs that recovery on a `card_amended: true` refusal.
- `lyx webster rebaseline` accepts a `00-overview.md` change confined to its Card Index, so a card added after the last begun batch lands; its refusals for a done batch's card and for any other overview change name that follow-up-card landing.
- webster's `doc.go` documents the sequence for an operator decision mid-Webster.
- Plan-Write's `archive-*/` rotation archives the Webster run record with the plan it retires.
- The begin-batch existing-report refusal names the one remedy for the recorded state; a refusal from any webster verb whose config content fails to parse or validate carries `config_invalid: true` and Master's ladder stops on it.
- Loom-Preflight validates `batcher.yaml` through `batcher.Active`.
- `lyx config reconcile` migrates retired `batcher.yaml` profile keys (`startup_context`, `alone_above`, `master_base`).
- A computed Merriam base in `websterengine`, passed to `Batch`; `master_base` gives way to an `orientation` weight in `Weights`, `template.yaml` and the calibrate fit in `tools/tokencount`.

Out:

- A combined verb that edits the decision record and the plan in one call; the existing `lyx loom decision add` and `lyx webster rebaseline` stay separate verbs.
- Amending `00-overview.md` mid-run outside its Card Index.
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
Every target keeps `PlanReset`'s existing guards (merge in progress, foreign branch, missing commit, target not an ancestor of HEAD, foreign dirty paths) and performs the reset through fabric's pair-checkout reset, except `start`'s archive-only path, which D3 bounds.

Bound, per new target:

- `report-head` reads a fork-written SHA, so it is accepted only when batch NN is begun and not terminal, its report is on disk, and the report head descends from (or equals) batch NN's recorded `StartSHA`; it can discard only commits above that batch's report head.
- `report-head` is the one target allowed while the run lock is held, because Master calls it inside the run, between the fork's report and record-batch; every other target keeps the run-lock refusal.
  Under a held lock it lets through a reset while Master's session is live, and three checks bound that: batch NN is the highest begun batch, so no later batch's fork has a legitimate start; no recovery strand of batch NN is live in reed, so the one writer lyx can see is absent; and the fork's report is on disk, which is the fork's last act.
  What stays unchecked is a Master that spawned a fork out of order: lyx has no handle on an in-session fork, so no check can see one, and that residue is named in Open risks.
- Like every target, `report-head` moves the remote task branch to the target through `ResetPairCode`, under that function's own bound: only the pair's task branch, only when every dropped commit is reachable from HEAD, with a lease on the remote tip.
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
When the start target cannot be moved to, it archives the record without moving the branch and reports `moved: false` with the reason in its envelope.
That archive-only set is closed: no batch recorded a start, a recorded start is missing from the repository, the recorded starts share no single oldest commit and no common ancestor, or the resolved start is not an ancestor of HEAD.
Each of these refuses today with `lyx webster run --fresh`, which D3 retires from the loom path, so none keeps a refusal.
The archive-only path keeps `PlanReset`'s refusals that come before the target is resolved: run lock held, no `state.json`, merge in progress, foreign branch.
It drops the missing-commit, no-common-ancestor, ancestor and dirty-tree refusals, which guard a branch move it does not make.
Like a moving reset to start, it removes the run's live recovery strands before archiving, and refuses with the `lyx reed remove` way forward when one cannot be removed, so no strand keeps writing against an archived record.
The next `lyx webster run`, plain or from the Webster producer with `Fresh: false`, then starts a new run.
Every way forward that today reads "`lyx webster reset --to start`, then `lyx webster run --fresh`" becomes "`lyx webster reset --to start`, then `lyx webster run`"; a refusal surfaced through the Webster producer keeps ending in the row's re-entry step.

Bound: the archive-only path runs no git and moves no commit; it moves the run record into an archive directory, never deletes it.
What it lets through is a branch rewritten under the run, and any uncommitted changes: the next run starts over that tree as it stands.
The tree a following run starts from is the reset tree plus whatever uncommitted changes the reset left, which D4 states per mode.
`run --fresh` stays as a CLI flag for a run outside loom.

Rationale: every reset to start is followed by a fresh run, and the producer can never run fresh, so the pair "reset, then `run --fresh`" had no path through loom.
Folding the archive into the reset gives one verb an orch or operator runs, then re-steps.

### D4 — Standalone reset

Decision: in standalone mode, `lyx webster reset` performs the planned reset with `git reset --keep <sha>` through `internal/gitrepo` (a keep-reset beside the existing hard reset, on the `gitexec` side of the client boundary), and archives the record for `--to start` as D3 does.
It touches no remote.
When `--keep` refuses on a dirty path, the verb surfaces git's refusal with a way forward per path: restore a path the run wrote (`git checkout -- <path>`), commit or restore a path it did not, then re-run.

Bound: standalone has no fabric gate, so the keep form is the guard: it refuses rather than discards uncommitted changes.
The two modes leave different trees behind, and that is accepted:

- Pair: the run's own tracked changes are discarded; foreign tracked changes refuse the reset; untracked files stay.
- Standalone: every uncommitted change git can carry across the move stays, the run's own included; untracked files stay.
- D3's archive-only path, either mode: nothing is discarded and no dirty path refuses; tracked changes, the run's own included, and untracked files stay.

So after any reset the next run may start over uncommitted work the earlier run left: untracked files in either mode, and tracked edits after a standalone or archive-only reset.
Every reset to start, in either mode, and every standalone reset lists the uncommitted paths it left in its envelope, so the operator sees them.

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
It restamps the card's hash as it does for a terminal failed batch, and records the card in a new per-batch `AmendedCards` list in `state.json`, each entry carrying a `rendered` flag, false when recorded.
The live attempt (fork or recovery strand) keeps running on the old text.
When that attempt's report is recorded (record-batch, or recovery's terminal persist) and the batch holds an entry with `rendered: false`, the batch runs every normal check and is then recorded failed with an added `card_amended` finding naming each such card, whatever the report's status.
The refusal that reports this failure carries `card_amended: true` beside `batch_failed: true`.
recover-batch treats `card_amended` as a recoverable finding (never uncheckable, never `needs_fresh`).
Every recovery strand it spawns while the batch holds any `AmendedCards` entry gets, in its failure digest, an instruction per amended card to re-read the card and bring the committed work in line; the digest is rendered from `AmendedCards`, not from the previous failure's findings, so a stuck, dead or failed recovery does not erase it.
Spawning the strand sets each entry's `rendered: true`.
A rebaseline naming a card that already has an entry sets that entry's `rendered` back to false, so a re-edit while a recovery runs on the earlier text forces that recovery's record failed and the next recovery renders the newer text.
A recovery whose report is recorded done clears the batch's `AmendedCards`; a recovery that ends stuck, dead or failed leaves them, so the next `recover-batch`, by Master or the operator, renders them again.
Master's ladder gains a rung: a `batch_failed` refusal carrying `card_amended: true` runs `recover-batch` for that batch even when the failed attempt was itself a recovery; every other recovery failure keeps the existing stop.

Bound: only the in-flight batch, only cards named with `--card`, never a change to a batch's card-ID set, and an overview change only as D7 bounds it.
Each amendment forces at most one failed record and so at most one extra recovery from Master's ladder, because only an entry with `rendered: false` forces the failure and the flag turns true at the first spawn that renders it.
A recovery that records done is trusted to have followed the amendment instruction, as for any card, under the same checks; the amendment is not re-verified beyond them.

Rationale: the operator's decision against a running card must land in the card itself so the plan and the implementation agree; recording the batch failed reuses the existing failed-batch recovery path instead of a new one.

### D7 — A decision against a done card or the overview

Decision: rebaseline accepts a `00-overview.md` change confined to its `## Card Index` section, the lines from that heading to the next `## ` heading, as `planparser` bounds the index.
Everything else in the file must be unchanged: the frontmatter, the framing above the index, and the plan-level sections after it, `## verify:` (the plan's integration verify), `## Shared Decisions` and `## Rename mechanic`.
To check that, the baseline records a second hash of `00-overview.md`, over the whole file with the Card Index section's lines removed, in a new `state.json` field beside `PlanFileHashes`, restamped wherever `PlanFileHashes` is; a state without it refuses any overview change as today.
The index change itself is held to rebaseline's existing card-set rule, which compares `NN-slug` ids: every begun batch keeps its recorded cards, so only cards after the last begun batch can be added, removed or reordered.
rebaseline keeps refusing an edit to a card of a terminal `done` batch, and to the overview outside its Card Index, and each refusal's way forward names the landing that works: restore the card or the overview, add a follow-up card after the last begun batch that carries the decision, with its Card Index line, then `lyx webster rebaseline --card NN` naming it.
The fingerprint-mismatch way forward begin-batch and run give for a changed overview changes the same way.

Bound: the accepted overview change can add, remove or reorder only index lines for cards no batch has begun; outside the Card Index nothing may change, and a done batch's commits are never reopened by a plan edit.
A begun card's index line keeps its number and slug; its one-line intent is not pinned, because the card-set rule compares ids only.
What that lets through is a reworded intent of a begun card in the batch table of a later Master render; the card file itself stays pinned by its recorded hash, so the work a batch is judged against cannot change through the index.

Rationale: a new card needs its Card Index line, because `planparser` builds the plan's cards from that index, and rebaseline today refuses every overview change, so the tail rebaseline claims to accept could never be reached on disk.
Reopening a done batch would rework code later cards may already build on; the follow-up card lands the decision without that risk.

### D8 — Decision record and plan together

Decision: the sequence for an operator decision mid-Webster is `lyx loom decision add` for the record, the card edit, then `lyx webster rebaseline --card NN`.
It is documented in webster's `doc.go` rebaseline section; webster texts name no loom verb.
D8 changes no refusal: the webster half is already named by the fingerprint-mismatch way forward begin-batch and run give for an unrebaselined card edit (`lyx webster rebaseline --card NN`), and after D6 no in-flight refusal remains to name it.

Rationale: both verbs exist and commit their own files; webster runs standalone too, so it stays loom-agnostic.

### D9 — A loom re-plan archives the Webster run record

Decision: when Plan-Write's stale-plan rotation moves the live plan into `archive-<stamp>/`, it also moves a present Webster run record into `archive-<stamp>/webster/` through `websterengine.ArchiveRunRecord`, told the destination, as PR-Rework does for a round.
The next Webster run starts a new run from the current HEAD, keeping the commits the old run made, since the new plan was written against that tree.
`PATTERN-plan-generation` is amended in the same commit: Webster's run record also moves with Plan-Write's `archive-*/` rotation.

Bound: the move runs only inside the rotation, only when a run record exists, and moves files without deleting or resetting anything.

Rationale: a `goto` back to Plan-Write retires the plan generation, and the run record is generation-scoped; leaving it behind made record-batch refuse `plan_drifted` and rebaseline refuse the begun card, a loop with no exit that kept the run's progress.

### D10 — record-batch config failure and begin-batch's remedy

Decision: a webster verb whose config fails to load (the wiring's `batcher.yaml` and webster config loads) refuses and changes no state; a re-run after the fix proceeds normally, so record-batch then records the batch.
The refusal carries `config_invalid: true` only when a present config file's content fails to parse or validate, a retired key included.
A config file that exists but cannot be read is a transient failure: its refusal carries no `config_invalid` and names a re-run.
The begin-batch existing-report refusal names one remedy, picked from the recorded state:

- Begun, not terminal, fork attempt: `lyx webster record-batch NN`, after fixing whatever its last refusal named.
- Begun, not terminal, recovery attempt: `lyx webster recover-batch NN`.
- Terminal `stuck` or `failed`: `lyx webster recover-batch NN`.
- Terminal `dead`: the batch's recovery is exhausted; end the run `stuck` naming the batch, as Master's dead rung does.
- Terminal `done`: the batch is finished; begin the next batch.

Master's ladder gains a rung: a refusal carrying `config_invalid: true` from any verb ends the run with `outcome: stuck`, quoting the refusal, without calling another verb.
Bound: the rung fires only on the parse-or-validation case above, so an unreadable file or any non-config refusal never stops a run through it.

Rationale: the issue's batch was already recoverable by re-running record-batch, but begin-batch listed several remedies and Master picked the wrong one; config is not Master's to fix.

### D11 — Loom-Preflight validates batcher.yaml

Decision: Loom-Preflight calls `batcher.Active` on the worktree's config, the same load the Batchifier row and webster's wiring make, and fails with a `config` reason naming `batcher.yaml` and the way forward.
When the failure is a retired key, the way forward is `lyx config reconcile --apply`, then re-step; otherwise it is the Batchifier row's existing fix-and-re-step text.

Rationale: the drift surfaced five rows into the run; Loom-Preflight is the first row that is loom's own, and `batcher` is a loom concern, not the general Preflight's.

### D12 — Reconcile migrates retired batcher keys

Decision: `configreg.Module` gains a migration hook that `lyx config reconcile` runs; batcher's hook rewrites each `profiles` entry:

- drops the profile-level `alone_above`;
- drops `weights.startup_context`;
- drops `weights.master_base`, and adds `weights.orientation` at the template's value only when the profile carries no `orientation`; an operator's own `orientation` is kept.

Reconcile reports each rewrite as a `migrated` entry, in the dry run and with `--apply`.
`batcher.Active` keeps refusing a retired key, and its message names `lyx config reconcile --apply`.
webster's `batcherLoadError` names the same step for a retired key.

Bound: the hook rewrites only the keys a module declares retired, at their declared level, and adds only `orientation` where it is missing; every other open-map entry is still carried whole.
It changes the operator's file only under `--apply`, after the dry run has listed each rewrite, which is reconcile's existing contract for every module; the loader itself never rewrites or tolerates a retired key.

Rationale: `profiles` is an open map, which reconcile carries whole, so a retired key inside a profile survived every reconcile; retiring `master_base` in this task would otherwise break every seeded `batcher.yaml` on the first run after deploy.

### D13 — Computed Merriam base

Decision: a function in `websterengine` computes Merriam's fixed part at partition time: the worktree's `CLAUDE.md` (and `CLAUDE.local.md` when present), the rendered Master stencil with the inlined PATTERN overview, and `00-overview.md`, the one plan file Merriam reads up front; each is sized with the batcher's own size model (lines × `context_per_line`), plus a Go constant for the system prompt and tools.
`Batch` takes the computed base beside `before`, and `startContext` becomes `computedBase + orientation + (k−1) × batch_growth`.
`orientation` replaces `master_base` in `Weights`, `template.yaml` and the calibrate fit in `tools/tokencount`, which fits `orientation` against measured starts minus each run's computed base.

The numbers come from the board entry's measurements over three `cautious` runs: Merriam's session starts at about 20.6K, which already holds the system prompt, tools, `CLAUDE.md` and the rendered Master stencil, and its first fork starts at the fitted `master_base` of 52000.

- The Go constant is 20.6K minus the computed size of `CLAUDE.md` and the rendered Master stencil at this task's HEAD, so the computed base reproduces the measured start; the constant's comment names that derivation.
- `template.yaml`'s `orientation` is 31400, the measured 52000 minus the measured 20.6K start, which needs no old run data.
- When the calibrate fit runs over measured runs whose transcripts and plans are still on disk, its `orientation` replaces 31400; when they are gone, 31400 stands.
A breakdown recorded in an older `state.json` with `master_base` still decodes, read-only, as `startup_context` does today.
The identity batchifier ignores the base, as it ignores `before`.

Rationale: the base is fixed by what Merriam loads, so a bigger plan or another repo lands far from a fitted 52K; computing the loaded part leaves only the orientation work to fit.

## Constraints

- `PATTERN-refusal-way-forward`: every new or reworded refusal (reset targets, rebaseline's done-card and overview-framing refusals, begin-batch's remedy, `config_invalid`, `card_amended`, Loom-Preflight's batcher check) names its way forward, has its row in `contracts/specs/refusal-spec.md` in the same commit, and is reached by a test.
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
- In-session forks cannot be interrupted, so D6 lets the old attempt finish and re-runs the batch by recovery.
- Merriam reads only `00-overview.md` up front (its stencil says so), so the other plan files stay out of the computed base.
- The board entry's 20.6K start includes the rendered Master stencil and `CLAUDE.md`; D13's constant subtracts them on that premise, and drift in it is absorbed by refitting `orientation`.

## Open risks

- D6's extra recovery costs a recovery strand's full cold start per amendment.
- D3's archive-only path accepts a branch rewritten under the run without moving it; the operator sees `moved: false` and the reason, but nothing else stops them.
- `report-head` under the run lock cannot see an in-session fork: a Master that spawned a fork out of order, for batch NN after its report or for a batch it has not begun, could have that fork writing during the reset; D1's checks rule out every other writer.
- The computed base sizes text by lines, so a stencil with long lines prices low; the fit absorbs the bias only on average.

## Acceptance criteria

- `lyx webster reset --to report-head --batch NN` moves HEAD to the report's `head_sha` while the run lock is held, and refuses for a terminal batch, a missing report, a report head that does not descend from the batch's start, a later begun batch, and a live recovery strand of batch NN.
- `lyx webster reset --to last-batch-head` and `--to batch-start --batch NN` move HEAD to their recorded commits; `batch-start` refuses when a later batch recorded a start; both refuse under a held run lock.
- `--batch` is refused with `start`, `pre-fix` and `last-batch-head`, and required with `report-head` and `batch-start`.
- `lyx webster reset --to start` archives the run record so a following plain `lyx webster run` starts a new run and, in either mode, lists the uncommitted paths it left; with no recorded start, a missing start, starts with no common ancestor or a non-ancestor start it archives without moving and reports `moved: false`, still refuses under a held run lock, a merge in progress or a foreign branch, and removes live recovery strands first.
- No production Webster text contains `git reset --keep` or names `lyx webster run --fresh` as a step after a reset; a test pins the absence.
- Standalone `lyx webster reset` moves HEAD with the keep form, lists the uncommitted paths it left, and surfaces git's refusal on a dirty path it would overwrite with the restore step for a run-written path and commit-or-restore for any other.
- A recovery prompt rendered over a dirty worktree lists each uncommitted path under the keep or the leave instruction by the run's write evidence, and renders `none` on a clean tree.
- `lyx webster rebaseline --card NN` on an in-flight batch's card is accepted; the batch's next recorded report yields a failed batch with a `card_amended` finding and a `card_amended: true` refusal even when the report says done; recover-batch renders the amendment in the failure digest, does not answer `needs_fresh`, and marks it rendered, so that recovery's own report is not forced failed.
- After a recovery that rendered an amendment ends stuck, dead or failed, the next recover-batch still renders it; after a recovery records done, the batch holds no amendment; a rebaseline re-naming a card whose amendment was rendered forces the live recovery's record failed.
- Master's template runs `recover-batch` on a `batch_failed` refusal carrying `card_amended: true` after a recovery, and keeps stopping on every other recovery failure.
- rebaseline accepts an added card after the last begun batch with its Card Index line in `00-overview.md`, and refuses a change outside the Card Index (frontmatter, framing, `## verify:`, `## Shared Decisions`, `## Rename mechanic`), to a begun card's number or slug in the index, and to a done batch's card, naming the follow-up-card landing; a state without the second overview hash refuses every overview change.
- webster's `doc.go` rebaseline section documents the `lyx loom decision add`, card edit, `lyx webster rebaseline --card NN` sequence, and no webster production text names a loom verb.
- A loom `goto` to Plan-Write after a batch began leaves the old run record under the plan's `archive-*/webster/`, and the next Webster run starts a new run without a `plan_drifted` refusal.
- A webster verb whose `batcher.yaml` fails to parse or validate refuses with `config_invalid: true` and leaves `state.json` and the report unchanged; record-batch re-run after the fix records the batch; an unreadable `batcher.yaml` refuses without `config_invalid`.
- begin-batch's existing-report refusal names exactly one remedy for each recorded state in D10.
- Loom-Preflight fails on a `batcher.yaml` carrying `startup_context` or `master_base`, naming `lyx config reconcile --apply`.
- `lyx config reconcile` reports and, with `--apply`, performs the batcher migration, leaving every other profile entry intact; a profile carrying both `master_base` and `orientation` keeps its own `orientation`.
- `template.yaml` carries `orientation: 31400` unless the calibrate fit over still-available measured runs gives another value.
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
