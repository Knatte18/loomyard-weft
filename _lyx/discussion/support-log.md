# Support log — webster-dead-ends

## Interview

The session ran in autonomous mode: no operator answered, and every turn below is a self-posed question with its auto-picked answer, after reading the board entry and the code it names.

1. Scope check against the board.
   No other running task touches `websterengine`, `batcher` or `loomshed`'s Plan-Write and Loom-Preflight rows; `shuttle-state`, `reed-view` and `plan-glyph-precision` touch other modules.
   The notes `webster-parallel-execution` and `kick-start-pack` are parked and unaffected.
2. Which sites name `git reset --keep`?
   Reading showed four, not three: `reconcileHead` (record-batch and recover-batch, worded "the report's head_sha"), the same function for accept-audit ("the last batch head"), accept-audit `--batch`'s batch-start refusal, recover-batch's needs-fresh parenthetical, plus standalone reset.
   The batch-start case needs its own target, so the reset verb gets three new targets, not two.
3. Who reads the report-head refusal?
   Master, inside its run, while the run lock is held; `PlanReset` refuses under the run lock.
   A lyx target that kept that refusal would turn a working `git reset --keep` into a dead end, so `report-head` alone is allowed under the run lock, with a bound on the report head's ancestry.
4. Why does "reset, then `run --fresh`" dead-end in loom, and what closes it?
   The Webster producer runs `Fresh: false` by design.
   Every reset to start is followed by a fresh run, so the reset archives the record itself; the cases where no reset is possible archive without moving.
5. Standalone reset: refuse or perform?
   Perform with the keep form through `gitrepo`, since standalone has no fabric gate and the keep form refuses rather than discards.
6. Recovery prompt on a dirty tree: keep or discard?
   The run's transcript write evidence, already used by reset's dirt check, tells the earlier attempt's paths from foreign ones, so the instruction is per path: keep and check the run's own, leave foreign ones alone.
7. A decision against a begun card: which begun states?
   Terminal failed, dead and stuck already accept an edit.
   The in-flight batch cannot have its fork interrupted, so the edit is accepted and the batch is re-run by recovery once its attempt reports.
   A done batch is not reopened; a follow-up card carries the decision.
8. A loom `goto` to Plan-Write after a batch began.
   `PATTERN-plan-generation` already makes the Webster run record generation-scoped for PR-Rework; Plan-Write's own rotation left it behind, which is the #413 loop.
   The rotation now moves it too, and the PATTERN entry is amended.
9. Issue #441: does record-batch mutate state before its config loads?
   No: `batcher.yaml` loads in webstercli's wiring, before the verb runs, so the batch was already recoverable by re-running record-batch.
   The gap is begin-batch's multi-remedy message and Master's lack of a config rung.
10. Issue #440: Preflight or Loom-Preflight?
    The general Preflight is orchestrator-agnostic; batcher is loom's concern, so Loom-Preflight.
    The Batchifier row already gates `batcher.Active`, so the #440 run's config most likely changed under it, which is why the reconcile migration and the `config_invalid` stop are also needed.
11. Why migration rather than a clearer refusal only?
    `profiles` is an open map that reconcile carries whole, so a retired key inside it survives every reconcile, and this task retires `master_base` itself.
12. Computed base: which files and which unit?
    Merriam's stencil has it read only `00-overview.md` up front, so that, `CLAUDE.md` and the rendered Master stencil form the computed part, sized with the batcher's own line model; the system prompt and tools are a Go constant.

## Rejected alternatives

- Only `report-head` and `last-batch-head` targets, as the board names: leaves accept-audit `--batch` and recover-batch naming `git reset --keep` for the batch start.
- Raw `--sha` on reset: the verb's contract is a run-recorded commit only.
- Keeping the run-lock refusal for every target: Master's in-run report-head recovery would dead-end.
- A separate `discard-run` verb for the archive: one more verb for a step that always follows a reset to start.
- Letting the Webster producer run `Fresh: true` on a fingerprint mismatch: the producer comment keeps the destructive fresh run an explicit human act.
- Standalone reset that keeps refusing with a git command: the last webster text naming git.
- Recovery prompt that tells the agent to discard every uncommitted path: throws away the earlier attempt's work the run's own evidence vouches for.
- Recovery prompt that keeps every uncommitted path: lets foreign changes into the batch's commit.
- Interrupting the in-flight fork on an amendment: in-session forks are Master's subagents; lyx has no handle to stop them.
- Accepting the amendment and recording the old attempt as done: the plan and the code would disagree.
- Reopening a done batch on an amendment: later batches may build on its code.
- A combined `amend` verb writing the decision record and the card: crosses the loom and webster module boundary for a two-command sequence.
- Making the Webster producer rebaseline every changed card automatically after a re-plan: begun cards' code was written to the old text.
- Validating `batcher.yaml` in the general Preflight: Preflight is orchestrator-agnostic.
- Validating every config module at Loom-Preflight: the failure seen is batcher's; no other module has drifted.
- `batcher.Active` tolerating retired keys with a warning: `PATTERN-config-strictness` and the existing refusal keep a stale profile from being silently misread.
- Sizing the computed base in tokens through a tokenizer: a second size model beside the batcher's line model.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Which reset targets? — resolved, auto-pick: `report-head`, `last-batch-head`, `batch-start` (D1).
- Q2 Report-head under the run lock? — resolved, auto-pick: allowed, bounded by batch state and ancestry (D1).
- Q3 How does loom reach a fresh run? — resolved, auto-pick: `reset --to start` archives the record (D3).
- Q4 Standalone reset? — resolved, auto-pick: performed with the keep form through `gitrepo` (D4).
- Q5 Keep or discard a dirty tree at recovery? — resolved, auto-pick: per path by write evidence (D5).
- Q6 Amendment of an in-flight card? — resolved, auto-pick: accepted, batch re-run by recovery (D6).
- Q7 Amendment of a done card or the overview? — resolved, auto-pick: refused with a follow-up-card way forward (D7).
- Q8 Decision record and plan in one verb? — resolved, auto-pick: two existing verbs, documented sequence (D8).
- Q9 Re-plan after a batch began? — resolved, auto-pick: Plan-Write's rotation archives the run record; PATTERN-plan-generation amended (D9).
- Q10 Record-batch config failure? — resolved, auto-pick: `config_invalid` stop, single-remedy begin-batch refusal (D10).
- Q11 Where to validate `batcher.yaml`? — resolved, auto-pick: Loom-Preflight (D11).
- Q12 Migrate or report retired keys? — resolved, auto-pick: both, reconcile migration hook plus naming it in refusals (D12).
- Q13 Computed base inputs and unit? — resolved, auto-pick: `CLAUDE.md`, rendered Master stencil, `00-overview.md`, line model, Go constant for system prompt and tools (D13).
- Q14 Source of the `orientation` template value? — open for Plan: the calibrate fit over existing measured runs, else derived from 52000 minus their computed base.
