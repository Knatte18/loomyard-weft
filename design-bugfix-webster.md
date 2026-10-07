# Bugfix: Webster and plan validity

`bugfix-webster` · Task · bug, webster, loom, quarry · running · Batchifier

Bugfix only (operator, 2026-10-06): fix what each section below reports, with a regression test per fix that drives the real CLI wiring, not only fakes.
No new verbs, config keys or features; a reviewer treats any extension as a scope finding.
The board adds more bug sections here until the task starts.

Scope limits: from `webster-delete-ordering`, only the Plan-Gate refusal of backward dependencies and cycles; Webster dropping its own sequencing is not part of this task.

## webster-delete-already-gone

Seen 2026-10-03 in the `verify-publish-hardening` run, after 12 batches.
Batch 15 (card 15, `webster-verify-gate`) deleted `internal/websterengine/integration_test.go`, even though card 15 does not list it.
Card 17 lists the file under **Delete:**, so batch 16's `begin-batch` refused with `plan_drifted`: the path does not exist on disk and is not a Create target or Rename destination of any card.
The run blocked until the plan was edited by hand and rebaselined.

A Delete target that is already gone is the state the card wants, so it should not block the run.
Options:
- `begin-batch` accepts a missing **Delete:** path when an earlier batch's commit deleted it, and notes it in the batch record.
- The per-card check flags a batch that deletes a file outside its card, at the batch where it happens rather than at a later card.


From GitHub #352 (intake 2026-10-04).

## webster-delete-ordering

### Rule (operator, 2026-10-06)

A valid plan is executable strictly in card-number order: implementing card 1, then 2, then 3 and so on always works.
A card never depends on anything a later card does.
Webster follows that order as written and never sorts cards.

This reverses `contracts/specs/loom-plan-spec.md`, which today calls the card list a DAG of intent "not itself an execution order" and lets the executor derive the order (`websterengine.SequenceBatches`, `deriveEdges` in `internal/websterengine/sequence.go`).

### What changes

- **Plan-Gate refuses a backward dependency.** A card whose `Uses:` names a target of a later card, a card that deletes a symbol still referenced from a member a later card edits, and any derived dependency cycle are blocking findings, so an invalid plan never reaches Webster.
- **Webster drops its own ordering.** Batches run in card-number order; `SequenceBatches` and its cycle condensation go, or reduce to an assertion that the plan already satisfies every edge.
- **The plan spec and the plan template** state the rule to the plan writer and the plan reviewer.

### Sightings

- 2026-10-03, `pair-lifecycle` (GitHub #350): card 12 deleted `fabricengine.Topology.RemovePairBranch`, `PairBranchResult` and `PairSiblingRemnant` and ran before card 11, which removed their only production caller in `internal/battencli/wire.go`; the batch failed and the run blocked at Webster.
- 2026-10-06, `skills-in-one-turn`: card 2's `Uses:` named `internal/shuttleengine#Runner.skillLoader`, which card 4 edits, a backward dependency; cards 2 and 4 also shared test-file targets.
  `SequenceBatches` condensed the cycle {2, 4} and ran 01, 02, 04, 03, so card 4 deleted `SkillLoader.SkillLoadSequence`, `SkillUnknown` and `Runner.LoadSkill` before card 3 moved `internal/orchengine` and `internal/orchcli` off them.
  The cycle surfaced only as an unread `Cycle` warning.
  `webster rebaseline` refused the fix because batch 4 had begun, so the run needed `webster reset --to start`, an archived `state.json` and a goto, redoing two finished batches.

## webster-background-shell-wait

Imported from [issue #397](https://github.com/Knatte18/loomyard/issues/397).

### What happened

`lyx webster` wrote a `webster-background-shell` note titled "Master's wait counted a turn end past a background shell".
It lists a background shell, a polling `until grep -q '^done$' <log>; do sleep 5; done` loop, that had run past `background_shell_wait_min` (10 minutes) while Master's turn end was being judged.
The note says the shell may still be running in the session.

### Why it is abnormal

The note records that Master's turn end was counted although a background shell was still alive past the wait limit.
It is unclear whether the turn end should have been held until the shell finished, or whether the shell should have been stopped.
A shell left running across a turn end can outlive its purpose or race with the next step, and the note gives the operator no outcome: whether the shell was killed, whether the wait gave up, and what the run did next.

### What to do

Decide the intended behavior when `background_shell_wait_min` elapses with a shell still running: keep waiting, kill the shell, or escalate.
Make the note state which of these happened.

Provenance: task `test-suite-measure`; notes: `webster-background-shell.md`.

## webster-batch-validation-speed

Seen on `test-suite-change-amplification` (2026-10-03), a 45-card plan that the Batchifier split into one card per batch.

### Measured

- `lyx webster validate` on that plan takes 64 s of wall time at 100 % of one core, with no child processes: the time is spent in-process, single-threaded.
- `begin-batch` runs `planglyph.ValidateDispatch` over the whole plan before every batch (`internal/websterengine/beginbatch.go:303`), so each batch pays roughly that cost again.
- The gap from one batch's report to the next batch's spawn was 143 s, 72 s and 81 s for the first three batches, while the forks themselves took 1–2 min each.
  Over 45 batches that is on the order of 50 minutes of setup.

### Cause

Not loomyard's own code: `quarry.Repo.Resolve` re-parses every file in a target's directory for each file target (Knatte18/quarry#34).

### Per-batch validation stays (operator)

Validating before every batch is deliberate.
A fork may drift from the plan when it finds something the plan missed, and the plan changes with it; the per-batch check is what notices.
It is also the base for running forks in parallel later: glyphs, quarry and the validator maintain, at every batch, a DAG that decides which cards and batches can run concurrently.

### Want

- quarry#34 fixed and the quarry dependency bumped, so one validation of a plan this size takes seconds.
- If that is not enough, make the per-batch validation incremental: re-resolve only the targets in files the last batch's delta touched, plus the cards that reference them, and reuse the rest from the previous batch.
- Drop `begin-batch`'s per-batch model assertion (operator): the fork inherits Merriam's model, so `begin-batch` has no model to check.

### Related

The one-card-per-batch split makes the per-batch cost dominate; whether the Batchifier should group cards more is a separate question.


From GitHub #345 (intake 2026-10-04).


## From issue #401

Imported from [issue #401](https://github.com/Knatte18/loomyard/issues/401).

## What happened

A loom run passed Plan-Write, Plan-Bouncer and Batchifier with a batch order in which batch 04 (a delete-only card) ran before batch 03.
Batch 04 deletes symbols that batch 03 still uses, so the delete cannot be done without doing batch 03's work.
The agent assigned to batch 04 noticed this, stopped without editing anything and wrote a friction note.
`lyx webster record-batch 04` then refused with `batch_failed` and ten `delete-not-done/...[blocking]` findings ("Delete target ... still resolves").
The refusal output ends with `way forward: lyx webster recover-batch 04`.
`lyx webster recover-batch 04` refused with the same ten findings and the same `way forward: lyx webster recover-batch 04`.
The run halted `blocked` at Webster with anomaly `escalation-to-human` (history ends `Webster / stuck`).

## Why it is abnormal

- The plan and batch order carried a dependency inversion that neither Plan-Bouncer nor Batchifier caught, although the card's own text referred to the other card as already done.
  Batch ordering has no dependency check, or it ignores the dependency the plan states.
- The refusal's `way forward` points at a verb that cannot succeed.
  `recover-batch` re-runs the same `delete-not-done` check against an unchanged tree, so following the hint loops.
  The real cause (batch order) is not named in the refusal.

## What to do

- Have Plan-Bouncer or Batchifier reject or reorder a batch whose delete targets are still referenced by a later batch, or that depends on a batch ordered after it.
- When `delete-not-done` is blocking and a later batch still references the target, say so in the refusal, and do not suggest `recover-batch` as the way forward when nothing has changed since the failure.

Provenance: task `skills-in-one-turn`; notes `04-drop-per-skill-load.md`, `loom-halt.md`, `webster-refusal-record-batch.md`, `webster-refusal-recover-batch.md`.


## From issue #402

Imported from [issue #402](https://github.com/Knatte18/loomyard/issues/402).

What happened:
In one loom run, for three separate batches, `lyx webster begin-batch <n>` was refused because a report for the batch already existed under `_lyx/webster/reports/`.
The refusal text says a report left behind by a crashed session is consumed by `lyx webster record-batch <n>`.
`lyx webster record-batch <n>` was then refused for the same batch: "record-batch called with no begin-batch record for this batch".
It archived the report with a timestamp suffix and told the caller to run `lyx webster begin-batch <nn>` again.

Why it is abnormal:
The two verbs' guidance points at each other.
begin-batch says "record the existing report", and record-batch says "no begin-batch record exists, begin again".
Only the archive side effect of the failed record-batch breaks the loop, which makes the recovery path indirect and costs two refused calls per batch.
It happened for every batch in the run, so it looks like a systematic ordering problem (a report being present before begin-batch recorded its start), not a one-off crash.

What to do:
Find what writes the batch report before the begin-batch record exists.
Make begin-batch either adopt or archive a report that has no begin-batch record, so the refusal message names a path that works the first time.
The webster-refusal notes also don't say which of the two states the caller was in, so the refusal could include it.

Provenance: task skills-in-one-turn; notes webster-refusal-begin-batch.md, webster-refusal-begin-batch-2.md, webster-refusal-begin-batch-3.md, webster-refusal-record-batch.md, webster-refusal-record-batch-2.md, webster-refusal-record-batch-3.md.


## From issue #403

Imported from [issue #403](https://github.com/Knatte18/loomyard/issues/403).

What happened:
`lyx webster rebaseline --card=04` was refused: "the edited plan changes the cards of a batch this run already begun: batch 4 card ... changed since it was begun".
The only way forward offered was to restore the cards in the plan, or `lyx webster reset --to start` followed by `lyx webster run --fresh`.

Why it is abnormal:
The `--card` flag implies a targeted rebaseline of one card.
The refusal then only allows discarding the whole run's progress, which is disproportionate for a one-card edit and is not a path an agent can take on its own mid-run.

What to do:
Either let rebaseline accept an intentional edit to a begun card when `--card` names it (resetting only that batch), or have the refusal explain why `--card` cannot do so and offer a narrower way forward such as `lyx webster recover-batch`.

Provenance: task skills-in-one-turn; note webster-refusal-rebaseline.md.


## From issue #404

Imported from [issue #404](https://github.com/Knatte18/loomyard/issues/404).

What happened:
The loom run halted `blocked` at the Publish producer with anomaly `escalation-to-human` and reason "push rejected by the remote".
The trace shows `landingshed: producer stuck` with error "gitrepo: push rejected (remote diverged)", right after "post-merge verify skipped because the tree is already verified".
The run's earlier steps (Webster review, Describe) were all done.
The step ended with `state=blocked next=Publish`.

Why it is abnormal:
A diverged remote at push time is a routine race when other runs or pushes land on the parent branch.
Halting for a human at the last step of a finished run throws away no work but needs manual intervention for a condition lyx could handle.
It is unclear from the halt note whether Publish re-fetched and re-integrated before giving up.

What to do:
Check whether landingshed's Publish refetches, re-integrates and re-verifies once on a diverged push before escalating.
If it does not, add a bounded retry, and make the halt note say what Publish tried and what the operator should run to resume.

Provenance: task skills-in-one-turn; note loom-halt.md.
