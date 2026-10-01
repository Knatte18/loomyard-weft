# Every webster, shed and loom refusal names a way forward

Every refusal reachable through `lyx webster`, `lyx shed` and `lyx loom` now leaves a way forward that needs no hub code fix and no hand-edited state.
A guard that does not protect correctness warns and records instead of halting.
The 2026-09-30 wedges (`operator-surface`, `batten-hub-ready`, `ide-hub-workspace`) each needed a code fix and deploy to continue; those paths are closed and covered by regression tests.

The board entry for this task links no GitHub issue, so there is no `Closes` line.

## What changes

- **Refusal table.**
  New `contracts/specs/refusal-spec.md` lists every in-scope refusal per module (webster, shed, loom) with trigger, class and way forward.
  The package docs, `webster-spec.md` and the ly-drive skill link to it.
- **New invariant.**
  `CONSTRAINTS.md` gains the Refusal Way-Forward Invariant: a refusal in a bound module names its way forward and has a table row plus a reaching test in the same commit.
  The Shed Verb-Set list gains `goto`.
- **Fork audit split.**
  `fork-contract-write` and `parent-write` into the warp's tracked tree or under `_lyx` are correctness findings.
  `fabric-reference`, `named-spawn`, `nested-agent` and other `parent-write` paths are policy findings.
- **Policy findings warn.**
  On an `OK` report with only policy findings, record-batch re-runs the batch's cards' `verify:` commands in-process.
  When all pass, the batch records done and each finding becomes a warning, once per finding identity.
  A warning goes into `state.json`, onto the verb's envelope under `warnings`, and into `summary.md` under `## Audit warnings`.
  A failing verify treats the findings as correctness class.
- **No more refusal rings.**
  An `OK` report rejected on its merits (correctness finding, failed done-check, blocking handle-bind finding) now archives the report and records the batch failed, so `lyx webster recover-batch NN` proceeds.
  An unattributable report (no begin record, or no fork transcripts after the settle window) is archived and the message names `lyx webster begin-batch NN`.
- **Findings are dispositioned once.**
  A parent-transcript finding is consumed by the first record-batch that reports it.
  The run-exit cross-check passes findings already dispositioned.
  A leftover policy finding becomes a run-level warning.
  A leftover correctness finding demotes Master's `done` to `stuck`, naming the suspect paths to revert with `git` before re-stepping.
- **Later-card findings.**
  A drift or deleted-symbol finding about a card outside the batch being recorded is a warning on the current batch.
  That card's own begin-batch validation refuses it.
- **`lyx webster rebaseline`.**
  Accepts the on-disk plan: restamps the fingerprint in `state.json` and keeps every batch record.
  It refuses when a begun or terminal batch number would cover a different card set, and names restoring the cards or `--fresh` after a branch reset.
  `ErrFingerprintMismatch` messages now name `rebaseline` first.
- **`lyx shed goto [<run-id>] --to <producer>`.**
  Generic verb in `internal/shedverbs`, so the shed, loom and batten subtrees all gain it.
  It parks the run `paused` at the named row and appends a history entry with the new outcome `goto`.
  A `goto` entry resets the bounce budget for every producer in the target's `Segment`.
  It refuses while the run lock is held, for an unknown producer (listing valid names), and on a `done` run.

## Design decisions to review

- `goto` outcome is history-only: `internal/shedengine` declares it, no producer returns it, and loom's seed-coherence check accepts it.
  No new `step` refusal kind and no transient mark.
- `goto` writes `paused`, not `running`, so loom's entry observation does not read it as a crashed driver.
- `goto` places no restriction on targets.
  An entry gate that refuses a re-entered run is an ordinary table row; `Loom-Preflight`'s fresh-start check names `goto` to the next row.
- Hand-editing a lyx-owned state file or a code fix plus deploy never counts as a way forward; a plain `git` command on the warp does.
- `websterengine` stays fabric-blind; fabric commits of `state.json` and archived reports happen in `internal/webstercli`.
- Out of scope: landing, batten, orch, fabric, burler and board refusals, which need their own audit.

## Risks

- The table is only as complete as the enumeration; a missed refusal keeps its current behaviour until hit.
- After a correctness-class failure the recovery strand continues from the fork's committed tree and repairs rather than reverts.
  A suspect path it keeps is not re-audited.
- Re-driving an unattributable batch relies on begin-batch's begun-card validation scoping staying correct.

## To check by hand

- Run `go test ./...` and `go test -tags integration ./...`; the run's integration suite reported `OK` at HEAD `ca97012fac00812c6f1766011787806529386230`.
- `lyx webster status` lists a terminal `stuck` entry numbered -1 with slug `unknown` outside the card list; it did not block the run, but confirm it is not a stale record.
- Drive a real loom run through `lyx shed goto` and a foreign plan edit with `lyx webster rebaseline`; these were tested on fixtures only.
