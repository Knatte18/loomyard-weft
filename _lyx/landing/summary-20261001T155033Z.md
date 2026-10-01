# Every webster, shed and loom refusal leaves a way forward

A false stop is as serious as a wrong landing.
On 2026-09-30 three runs wedged on refusals whose only exit was a hub code fix and deploy:
a fork-audit false positive made a done batch unrecordable,
a begun-but-unrecorded batch was refused as create-already-exists,
and `record-batch`, `begin-batch` and `recover-batch` refused each other in a ring after a parent merge.
This change audits every refusal reachable through the `lyx webster`, `lyx shed` and `lyx loom` verbs, classifies each, and fixes the ones with no exit.

The board entry carries no issue number, so there is no `Closes` line.

## What changes

- **Refusal table.**
  New `contracts/specs/refusal-spec.md` lists each refusal with trigger, class and way forward, one section per module.
  Classes are correctness halt, policy guard, transient and wiring guard.
  The `websterengine` and `shedengine` package docs, `webster-spec.md` and the `ly-drive` skill link to it.
- **New invariant.**
  CONSTRAINTS.md gains the Refusal Way-Forward Invariant.
  A refusal in a bound module names its way forward and has a table row and a reaching test, and a guard that does not protect correctness warns instead of halting.
  The Shed Verb-Set list gains `goto`.
- **Fork audit split.**
  `fork-contract-write` and a `parent-write` into the warp's tracked tree or under `_lyx` are correctness class.
  `fabric-reference`, `named-spawn`, `nested-agent` and other `parent-write` paths are policy class.
- **Policy findings become warnings.**
  On an `OK` report with only policy findings, `record-batch` re-runs the cards' `verify:` commands.
  If all pass, the batch is recorded done and each finding is stored once in `state.json`, returned under the envelope's `warnings`, and appended to `summary.md` as `## Audit warnings`.
  A failing verify treats the findings as correctness class.
- **Rejected reports no longer wedge.**
  A correctness finding, failing done-check or blocking handle-bind finding now archives the report, marks the batch failed with a digest naming suspect `parent-write` paths, and points to `lyx webster recover-batch NN`.
  A report with no begin record, or with no fork transcripts after the settle window, is archived and points to `begin-batch NN`.
- **Findings are dispositioned once.**
  A parent finding is consumed by the first `record-batch` that reports it.
  The run-exit audit passes findings already dispositioned.
  Undispositioned policy findings become run-level warnings, and correctness findings demote `done` to `stuck` naming the paths to revert with `git`.
- **Later-card findings.**
  Drift or deleted-symbol findings about a card outside the batch being recorded warn instead of failing it.
  That card's own `begin-batch` refuses it.
- **`lyx webster rebaseline`.**
  Accepts the edited on-disk plan by restamping the fingerprint and keeping batch records.
  It refuses when a begun or terminal batch would cover a different card set, and names restoring the cards or `run --fresh` after a branch reset.
  Fingerprint-mismatch messages now name it first.
- **`lyx shed goto [<run-id>] --to <producer>`.**
  A generic verb in `internal/shedverbs`, so the shed, loom and batten subtrees all gain it.
  It parks the run `paused` at the target and writes a `goto` history entry.
  That entry starts a fresh bounce budget for the target's segment.
  It refuses while the run lock is held, for an unknown producer (the message lists valid names), and on a done run.
  Loom's seed-coherence check accepts the `goto` outcome, so the next `lyx loom run` files no crash-resume anomaly.
- **Gate messages.**
  Refusals in `loomshed`, `loomengine`, `loomcli`, `preflightshed` and the Webster adapter in `shedadapters` now name a way forward.
  The `Loom-Preflight` half-finished check names `goto` to the next row.

## Design decisions for review

- A way forward is a `lyx` verb, a state transition the run takes itself, or a plain `git` command on the warp worktree.
  Hand-editing `state.json`, `status.json`, `seed.json` or reports never counts.
- `goto` places no restriction on targets and does not re-open a done run.
  It is an operator and orchestrator verb, which `ly-drive` names in escalations but never calls on its own.
  It adds no `step` refusal kind, and the closed kind vocabulary and envelope keys are unchanged.
- `websterengine` stays fabric-blind.
  Fabric commits of `state.json` and archived reports happen in `internal/webstercli`.
- A consumed correctness-class `parent-write` is not re-audited.
  If the recovery strand keeps a path the digest names, that content can land.
- Recovery after a failed batch continues from the fork's committed tree rather than reverting it.
- The table is only as complete as the enumeration, so a missed refusal keeps its current behaviour until hit.
- Out of scope: refusals in landing, batten, orch, fabric, burler and board verbs, which get their own audit.

## Check by hand

- A live loom run through a `goto` re-entry and a `rebaseline` mid-run, since the tests use hub fixtures.
- The stencil copies on the board have drifted from the source, as the CLI warns (`class=source-ahead`).
  The new webster stencil text reaches runs only after a production deploy by the operator.
