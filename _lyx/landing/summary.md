# Every webster, shed and loom refusal names a way forward

A false stop is as costly as a wrong landing.
On 2026-09-30 three runs (`operator-surface`, `batten-hub-ready`, `ide-hub-workspace`) each wedged on a refusal whose only way out was a hub code fix and deploy, or hand-edited state.
This change audits every refusal reachable through the `lyx webster`, `lyx shed` and `lyx loom` verbs, fixes each one without a way forward, and records the result in a durable table.

## What changes

- **Refusal table.** New `contracts/specs/refusal-spec.md`, one section per module (webster, shed, loom), with columns Refusal, Trigger, Class, Way forward.
  The `websterengine` and `shedengine` package docs, the loom gate docs, `webster-spec.md` and the ly-drive skill link to it instead of restating rows.
- **Fork audit split by severity.**
  `fork-contract-write`, and `parent-write` into the warp's tracked tree or under the run's `_lyx` content, are correctness findings.
  `fabric-reference`, `named-spawn`, `nested-agent` and every other `parent-write` are policy findings.
- **Policy findings warn instead of halting.**
  On an `OK` report with only policy findings, `record-batch` re-runs the batch's cards' `verify:` commands in-process.
  If all pass, the batch records done and each finding becomes a warning, stored once per finding in `state.json`, returned in the envelope's `warnings`, and appended to `summary.md` as `## Audit warnings`.
  If any fails, the findings are treated as correctness findings.
- **No more refusal ring.**
  An `OK` report rejected on its merits (correctness finding, failing done-check, blocking bind finding) now archives the report and fails the batch, so `recover-batch NN` proceeds.
  An unattributable report (no begin record, or no fork transcripts after the settle window) is archived and the batch re-driven with `begin-batch NN`.
- **Each finding is dispositioned once.**
  Parent-transcript findings are consumed by the first `record-batch` that reports them, and fork transcripts are marked seen whether they passed, warned or failed.
  The run-exit audit passes findings already dispositioned; a policy finding it first sees becomes a run-level warning, and a correctness finding demotes Master's `done` to `stuck` with the suspect paths named.
- **A finding about a later card** becomes a warning on the current batch; that card's own `begin-batch` refuses it.
- **New `lyx webster rebaseline`** accepts the on-disk plan as the run's plan: it restamps the fingerprint and keeps batch records.
  It refuses when a begun or terminal batch would cover a different card set.
  Fingerprint-mismatch messages now name it first.
- **New `lyx webster accept-audit`** clears run-exit correctness findings left pending, once the named paths are restored with `git`.
  It checks each suspect path against the last batch head, refuses while any differs, and only edits `state.json`.
- **New `lyx shed goto [<run-id>] --to <producer>`**, in `internal/shedverbs`, so the shed, loom and batten subtrees all gain it.
  It sets the current producer to the named row, state `paused`, with a history entry of the new outcome `goto`.
  The bounce budget treats `goto` as an episode boundary for the target's whole `Segment`.
  It refuses while the run lock is held, for an unknown producer (listing valid names), and on a `done` run.
- **Entry gates** in the loom, preflight and Webster-adapter paths gained way-forward messages; `Loom-Preflight`'s fresh-start check names `goto` the next row.
- **CONSTRAINTS.md** gains the Refusal Way-Forward Invariant, and the Shed Verb-Set list gains `goto`.

Closes the board task `refusal-audit`.

## Design decisions to review

- A refusal has a way forward when it names a `lyx` verb, a state transition the run takes by itself, or a plain `git` command on the warp.
  Hand-editing lyx-owned state, or a code fix plus deploy, never counts.
- Every refusal is classed correctness halt, policy guard, transient or wiring guard.
  Wiring guards and raw I/O failures are grouped rows with no per-row test.
- The correctness rule for `parent-write` is checked first and wins, since `_lyx` is itself excluded from warp git.
- `goto` places no restriction on targets and does not reopen a `done` run.
  It sets `paused`, because a `running` write with the lock free reads as a crashed driver to loom's entry observation.
- `internal/shedengine` declares the `goto` outcome and no producer returns it; loom's seed-coherence check accepts it.
  The step refusal-kind vocabulary and envelope keys are unchanged.
- `websterengine` stays fabric-blind; fabric commits of `state.json` and archived reports stay in `internal/webstercli`.
- Out of scope: landing, batten, orch, fabric, burler and board refusals.
  They need their own audit, so those refusals may still wedge runs.

## Known risks

- The audit is as complete as the enumeration: a refusal it missed keeps its old behaviour.
- A consumed correctness-class `parent-write` is not re-audited; if recovery keeps a suspect path the digest names, that content can land.
- A re-driven batch (unattributable report) may already have committed cards; the new fork relies on `begin-batch`'s begun-card validation to find the work done.

## To check by hand

- `go test ./...` was not re-run for this description; run it, including the real-git fixtures.
- Exercise the three 2026-09-30 incidents against a dev build (`./deploy-dev`): a fork-audit false positive, a begun-but-unrecorded batch at run entry, and the parent-merge ring.
- Run `lyx shed goto` against a live loom run and confirm the next `lyx loom run` files no crash-resume anomaly.
- `lyx board get` warned that the board copies of `webster-template-master` and `webster-prefix-recovery` have drifted from the worktree source; those stencils changed here, so a production deploy (`update-plugins.sh`) is needed for the new text to take effect.
