# Webster dead ends closed: reset targets, config_invalid, amended cards

All five batches (cards 1–24) recorded `done`, each on the first fork; no recovery ran.
The head after the last batch is `cc9abf9cd`.

## Batch 01 — configengine-invalid-mark (cards 1–5), head `16a4d219d`

Added the `configengine.ErrInvalid` sentinel and `MarkInvalid` for a present config file's content failures.
Replaced `master_base` with a computed `StartBase` plus `orientation` in the batcher, retired the old keys, and added the batcher config migration.
`lyx config reconcile` now runs a module's migration, and Loom-Preflight fails a stale `batcher.yaml`.
Reported deviations: `tools/tokencount/main.go` and `internal/loomrecipe/resume_test.go`.

## Batch 06 — webster-config-invalid (cards 6–10), head `f52353eb7`

Webster refuses a broken config with `config_invalid` and Master stops on it.
`begin-batch`'s existing-report refusal names one remedy.
Merriam's start base is computed and passed to the batchifier, the orientation fit is measured against starts minus that base, and `gitrepo` gained the keep-reset.
Reported deviation: `internal/webstercli/cli_test.go`.

## Batch 11 — reset-targets (cards 11–16), head `98a42f9cc`

Added the `report-head`, `last-batch-head` and `batch-start` reset targets.
The archive and pending-findings guard of `run --fresh` were factored into the reset's entry, and one exported function lists the uncommitted paths minus the run's own state.
`reset --to start` archives the run record, archiving without moving when it cannot move.
The standalone reset performs the keep form itself, and every `git reset --keep` way forward is repointed at a reset target.
Reported deviations: `internal/websterengine/archivereset_test.go`, `runlevel_test.go` and `findingtext_test.go`.

## Batch 17 — retire-fresh-after-reset (cards 17–19), head `c2160a294`

Every reset route ends in a plain `lyx webster run`, with a scan test pinning the absence of the retired step.
A recovery strand is told its uncommitted paths and what to do with each.
A rebaseline of an in-flight batch's card is accepted as an amendment.
Reported deviations: `internal/websterengine/archive.go` and `contractevidence.go`.

## Batch 20 — amended-cards-recovery (cards 20–24), head `cc9abf9cd`

An amended batch re-runs through recovery, and `planparser` exposes the overview minus its Card Index.
Rebaseline accepts an index-only overview change and names the follow-up-card landing.
Plan-Write's rotation archives the Webster run record, and the plan commit includes an archived record.
Reported deviations: `internal/webstercli/recordbatch.go`, `internal/websterengine/rebaseline.go`, `internal/websterengine/planbaseline_test.go` and `internal/loomcli/wiring.go`.
