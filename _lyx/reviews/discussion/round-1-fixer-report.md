# Fixer report — discussion round 1

Every finding was checked against the code and fixed in `_lyx/discussion/decision-record.md` and `_lyx/discussion/support-log.md`.

## Fixed

- **F1** (BLOCKING): confirmed against `internal/planparser/parse.go` (cards come from the Card Index) and `internal/websterengine/rebaseline.go` (every `00-overview.md` change refused).
  D7 now has rebaseline accept an overview change confined to the `## Card Index`, checked by a new framing-only hash in `state.json`, with the existing card-set rule bounding the index change.
  Updated the Scope "In" and "Out" lines, D6's bound, the fingerprint-mismatch way forward, the acceptance line, and support-log Q15 plus a rejected alternative.
- **F2**: D1 now states what `report-head` lets through under a held lock and three mechanical checks: batch NN is the highest begun batch, no live recovery strand of NN, report on disk.
  It names the residue no check can see, an in-session fork spawned out of order, and the remote-branch move through `ResetPairCode` with that function's own bound.
  Updated Open risks and the acceptance line.
- **F3**: `AmendedCards` entries now carry a `rendered` flag.
  Only an unrendered entry forces the `card_amended` failure, and spawning a recovery marks the entry rendered.
  The digest is rendered from `AmendedCards`, so a stuck, dead or failed recovery keeps it.
  The batch's `AmendedCards` clears only when a recovery records done.
  The refusal carries `card_amended: true`, which Master's new rung reads.
  Added acceptance lines for the recovery-after-recovery case and the Master rung.
- **F4**: D3's archive-only path keeps the run-lock, no-state, mid-merge and foreign-branch refusals, drops the move-only guards, and removes live recovery strands first; acceptance line updated.
- **F5**: D4 states the tree each mode leaves (pair discards the run's own tracked changes; standalone keeps every change git carries; untracked files stay in both).
  It accepts a dirty start in standalone, lists the uncommitted paths left in the envelope, and words the way forward as restore for run-written paths and commit-or-restore for any other.
  D3's Bound points at D4 for this.
- **F6**: confirmed against `webster-template-master.md` (the `dead` rung stops the run).
  D10 gives `dead` its own remedy, ending the run `stuck`, matching the template.
- **F7**: `config_invalid` is set only on a parse or validation failure of present content; an unreadable file is transient and refuses without it.
  Scope, D10's bound and acceptance updated.
- **F8**: confirmed against `internal/batcher/config.go` (`alone_above` at profile level) and `estimate.go` (`startup_context`, `master_base` under `weights`).
  D12 places each retired key at its level, keeps an operator's own `orientation`, and states that the migration rewrites the operator's file only under `--apply`, after the dry run lists each rewrite.
- **F9**: Scope gained D7 and D8 items, and D8 has an acceptance line.
  D8 now says it changes no refusal: the existing fingerprint-mismatch way forward (`beginbatch.go`) already names `rebaseline --card NN`, and D6 removes the in-flight refusal.
  The Constraints list no longer names an in-flight rebaseline refusal.
- **F10**: the board entry (`lyx board get` body) is the source: a 20.6K Merriam start that already holds the stencil and `CLAUDE.md`, and first forks at 52000.
  D13 derives the constant as 20.6K minus the computed size of `CLAUDE.md` and the stencil at HEAD, and sets `orientation` to 31400 (52000 − 20.6K), which needs no old run data; the calibrate fit replaces it only when measured runs are on disk.
  Q14 is resolved, the Open risk is removed, the Auto-mode assumption is restated as its premise, and an acceptance line is added.
- **F11**: the issue-run speculation moved out of Auto-mode assumptions into support-log Interview 10, after D10 and D12 were checked to already carry its requirements.
  The fallback rule went into D13 before the Open risk was removed (see F10).

## Disputed

None.

## Deferred

None.
