# Bugfix: Webster and plan validity

`bugfix-webster` · Task · bug, webster, loom, quarry

Depends on: [`test-suite-prune-core`](design-test-suite-prune-core.md)

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
