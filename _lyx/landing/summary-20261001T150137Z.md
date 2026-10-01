# Every webster, shed and loom refusal names a way forward

A false stop is as serious as a wrong landing.
On 2026-09-30 three runs (`operator-surface`, `batten-hub-ready`, `ide-hub-workspace`) each wedged on a refusal whose only exit was a hub code fix and deploy.
This change audits every refusal reachable through the `lyx webster`, `lyx shed` and `lyx loom` verbs, so no run needs that again.

## What changes

- **Refusal table.**
  `contracts/specs/refusal-spec.md` is new, with one section per module.
  Each row gives refusal, trigger, class and way forward; I/O and wiring guards are grouped rows.
  The `websterengine` and `shedengine` package docs, the loom gate docs and `webster-spec.md` link to it, and the ly-drive skill uses it to look up an escalation's way forward.
- **Fork audit split by class.**
  `fork-contract-write`, a `parent-write` into the warp's tracked tree or under `_lyx`, and a fabric reference whose command can change files are correctness class.
  `named-spawn`, `nested-agent`, other `parent-write` paths and read-only fabric references are policy class.
- **Policy findings warn.**
  When record-batch sees only policy findings on an `OK` report, it re-runs the batch's cards' `verify:` commands.
  If they pass, the batch is recorded done and each finding becomes a warning.
  The warning is stored once per finding in `state.json`, returned under the envelope's `warnings` key, and appended to `summary.md` as `## Audit warnings`.
  A failing verify treats the findings as correctness class.
- **Rejected reports fail the batch.**
  A correctness finding, a failing done-check or a blocking bind finding now archives the report and records the batch failed.
  The error names `lyx webster recover-batch NN`, which then proceeds.
  Before, an `OK` report stayed on disk and begin-batch, record-batch and recover-batch refused each other in a ring.
- **Unattributable reports are archived.**
  With no begin record or no fork transcripts after the settle window, the report is archived and the error names `lyx webster begin-batch NN`.
- **Each finding is dispositioned once.**
  A parent finding is consumed by the first record-batch that reports it, and fork transcripts are marked seen whether they passed or failed.
  The run-exit audit passes findings already dispositioned.
  Policy findings it still sees become run-level warnings.
  Correctness findings demote `done` to `stuck` and stay pending in state until `lyx webster accept-audit`, which also unblocks run entry (`ErrPendingAuditFindings`).
- **Later-card findings.**
  A drift or deleted-symbol finding about a later card is a warning on the current batch.
  That card's own begin-batch refuses it.
- **`lyx webster rebaseline`** accepts the on-disk plan after a foreign edit: it restamps the fingerprint and keeps every batch record.
  It refuses when a begun or terminal batch number now covers a different card set, or a begun card's content hash changed (begin-batch and recover-batch now record SHA-256 per card).
  Fingerprint-mismatch messages name `rebaseline` first and `--fresh` only with the branch-reset precondition.
- **`lyx shed goto [<run-id>] --to <producer>`** is a generic verb in `internal/shedverbs`, so shed, loom and batten all gain it.
  It parks the run `paused` at the named row and appends a history entry with the new history-only outcome `goto`.
  A `goto` entry resets the bounce budget for every producer in the target's segment.
  It refuses a held run lock, an unknown producer, a done run, a running run and targets past the reference row.
  Way-forward texts carry the run-id.
  Batten admits Worktree-Teardown only once the child run is done, through a `PreGoto` hook.
- **Loom entry gates.**
  Loom-Preflight accepts a deliberate `goto` re-entry and otherwise names `lyx loom goto --to Loom-Preflight`.
  Seed coherence accepts the `goto` outcome, so the next `lyx loom run` files no crash-resume anomaly.
  Approve's refusals name `git push`/`git pull` and a backward goto to Publish.
- **Constraints and docs.**
  CONSTRAINTS.md gains the Refusal Way-Forward Invariant and `goto` joins the Shed Verb-Set list.
  `docs/overview.md`, the websterengine package doc and the Master and recovery stencils follow.

## Design decisions to check

- A way forward is a `lyx` verb, a state transition the run takes itself, or a plain `git` command on the warp.
  Hand-editing a lyx-owned state file or a hub code fix never counts.
- `goto` places no restriction on targets beyond the refusals above.
  The step refusal-kind vocabulary and envelope keys are unchanged, and `goto` sets no transient mark.
- `shedengine` still imports only stdlib, `state` and `lock`; it declares `goto` and no producer returns it.
- Landing, batten, orch, fabric, burler and board refusals are out of scope and may still wedge runs until their own audit.
- Known risks: the audit covers the refusals found by enumeration, so a missed one keeps its old behaviour.
  A consumed correctness-class `parent-write` is not re-audited, so a recovery that keeps a suspect path named in the digest can land it.
  Re-driving a batch after archiving an unattributable report relies on begin-batch's begun-card validation scoping staying correct.

## To check by hand

- The refusal table's cells were written before the later fixes landed; spot-check them against the final messages in code.
- `go test ./...` and the integration-tagged tests reported OK at the branch head; no run against a live loom hub was done.
- `update-plugins.sh` has not run; the `lyx board get` logs warn that the board stencil copies for the Master and recovery templates drift from source until a production deploy.
