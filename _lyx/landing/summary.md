# Every webster, shed and loom refusal names a way forward

A false stop costs as much as a wrong landing.
Three runs on 2026-09-30 (`operator-surface`, `batten-hub-ready`, `ide-hub-workspace`) each needed a hub code fix and deploy to continue.
Their wedges were a fork-audit false positive on a done batch, a begun-but-unrecorded batch refused as create-already-exists, and the `record-batch`/`begin-batch`/`recover-batch` ring after a parent merge.
This change audits every refusal reachable through `lyx webster`, `lyx shed` and `lyx loom`, and fixes each one that had no way forward.
A guard that does not protect correctness now warns and records instead of halting.

No GitHub issue is attached to the board entry, so there is no `Closes` line.

## What changes

- **Refusal table.**
  `contracts/specs/refusal-spec.md` lists every refusal per module (webster, shed, loom) with trigger, class and way forward.
  The `websterengine` and `shedengine` package docs, `webster-spec.md` and the ly-drive skill link to it.
- **Fork audit split by class.**
  `fork-contract-write`, and `parent-write` into the warp's tracked tree or under the run's `_lyx` content, are correctness findings.
  `fabric-reference`, `named-spawn`, `nested-agent` and every other `parent-write` are policy findings.
- **Policy findings become warnings.**
  When `record-batch` finds only policy findings on an `OK` report, it re-runs the batch's cards' `verify:` commands in-process.
  If they pass, the batch is recorded done and each finding is a warning.
  The warning is stored once per finding in `state.json`, returned under the envelope's `warnings` key, and appended to `summary.md` as `## Audit warnings`.
- **A rejected report fails the batch instead of wedging it.**
  A correctness finding, a failing done-check or a blocking handle-bind finding now archives the report and records the batch failed, with the reasons as its digest.
  The error names `lyx webster recover-batch NN`.
  An unattributable report (no begin record, or no fork transcripts after the settle window) is archived and the error names `lyx webster begin-batch NN`.
- **Each finding is dispositioned once.**
  A finding is consumed by the first `record-batch` that reports it.
  The run-exit audit passes already-dispositioned findings.
  A leftover policy finding there becomes a warning.
  A leftover correctness finding demotes Master's `done` to `stuck`, naming the suspect paths to revert with `git` before re-stepping.
- **Later-card findings only warn.**
  Drift about a card outside the batch being recorded becomes a warning.
  That card's own `begin-batch` refuses it.
- **`lyx webster rebaseline`** accepts an edited on-disk plan: it restamps the fingerprint and keeps every batch record.
  It refuses when a begun or terminal batch's card set changed.
  `ErrFingerprintMismatch` messages name it first, and `--fresh` only with the branch-reset precondition.
- **`lyx shed goto [<run-id>] --to <producer>`**, in `internal/shedverbs`, so the shed, loom and batten subtrees all get it.
  It leaves the run `paused` at the named row and appends a history entry with the new history-only outcome `goto`.
  A `goto` entry starts a fresh bounce budget for the target's segment.
  It refuses while the run lock is held, for an unknown producer, and on a done run.
  The missing-producer and exhausted-budget stops, and `Loom-Preflight`'s half-finished stop, now name it.
- **Refusal sweep.**
  Refusal messages in the webster engine and CLI, shed, the loom CLI and the loom gates now carry a way forward, each with a test that reaches it.
- **`CONSTRAINTS.md`** gains the Refusal Way-Forward Invariant, and the Shed Verb-Set list gains `goto`.

## Decisions to check

- A way forward is a `lyx` verb, a state transition the run takes itself, or a plain `git` command on the warp.
  Hand-editing lyx-owned state never counts.
- `nested-agent` is policy class although the board named only `fabric-reference` and `named-spawn`.
- `parent-write` is split by path instead of classed whole.
- "Commits pass verify" is implemented as a re-run of the cards' own `verify:` commands.
- `goto` places no restriction on targets and does not re-open a done run.
- `goto` writes `paused`, not `running`: a `running` write with the lock free reads as a crashed driver to loom's entry observation.
- A consumed correctness-class `parent-write` is not re-audited.
  If the recovery strand keeps a path the failure digest names, that content can land.
- A failed batch's recovery continues on the fork's committed tree and repairs it instead of reverting it.
- A re-driven batch after an unattributable report relies on `begin-batch` finding the already-committed cards done.
- The enumeration is the audit: a refusal it missed keeps its old behaviour.
- Out of scope: landing, batten, orch, fabric and burler refusals, and the fast-forward merge-in limit (it already names a git recourse).
  Landing and batten refusals may still wedge runs until their own audit.

## To check by hand

- The integration suite passed in its run and reported no failure or triage items.
- Run a real loom run through `lyx shed goto` and `lyx webster rebaseline` on a live hub.
  The tests use `hubforge` fixtures, not a running loom.
- `lyx board get` warns that the deployed `webster-template-master` and `webster-prefix-recovery` stencils are behind the worktree source.
  The Master stencil gained the `batch_failed` and `report_archived` rungs, so a production deploy (`update-plugins.sh`) is needed before those rungs reach running Masters.
