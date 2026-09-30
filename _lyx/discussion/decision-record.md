# Decision record — awaiting-status-history-growth

## Goal

Stop a batten run's `status.json` from growing without bound while `Run-Shed` waits on an unbounded human hand-off,
and stop `Run-Shed` from telling the operator to re-approve on the poll right after it resumed an approved child.

Today every 30 s `Run-Shed` poll returns a budget-exempt Stuck, and `shedengine` appends one `history[]` entry per poll.
Counted Stucks are capped by `max_bounces`, but budget-exempt ones are not, so a multi-day approval wait writes tens of thousands of near-identical entries.
The fix covers every budget-exempt wait, including `Run-Shed`'s bounded driver-exit grace wait, not only the approval wait.
Separately, after `callAwaiting` resumes an approved child, the child's status stays `awaiting` until its driver re-steps,
and a poll in that window matches the approval-acted marker and reports "already acted on and the child is awaiting again; re-run `lyx loom approve`", which is wrong.

## Scope

In:

- `internal/shedengine`: fold a budget-exempt Stuck into the immediately preceding identical budget-exempt Stuck entry instead of appending a new one.
- `internal/shedengine`: two new optional `HistoryEntry` fields recording the fold.
- `internal/battenshed`: tell "resumed, child has not re-stepped yet" apart from "child is awaiting again" in `callAwaiting`, and report the former with its own reason.
- Docs: `docs/overview.md`'s batten `status` paragraph, which today says `Run-Shed`'s self-bounce appends one entry per poll; the `shedengine` and `battenshed` doc comments that describe history appends and the awaiting dispositions.

Out:

- Counted (non-exempt) Stuck entries, including `Run-Shed`'s running arm: they stay one entry per call, already bounded by `max_bounces` (1440 for `Run-Shed`).
- Retroactive compaction of history already written by an in-flight run.
- Any change to `battencli`'s status-envelope truncation (`recentHistory`, `history_truncated`), which stays as is.
- Any change to the batten `CommitStatus` seam's skip rule.

## Decisions

- **D1 — Fold, don't cap.**
  When the step's new entry is a budget-exempt Stuck and the current last history entry is also a budget-exempt Stuck with the same `producer`, the same `output` and the same `gate_attempts` (both nil, or equal values), `shedengine` updates that last entry in place instead of appending.
  Rationale: exempt Stucks are by definition unbudgeted waits, and the awaiting one is the only unbounded source of history growth; a fold keeps the file O(1) per wait episode and loses nothing but per-poll timestamps, since the entries were otherwise byte-identical.
- **D2 — The fold is engine-generic and automatic, keyed on `BudgetExempt`.**
  No new producer flag; any producer returning a budget-exempt Stuck gets folding.
  Rationale: `BudgetExempt` already declares "this is a wait, not an attempt", which is exactly the property that makes folding safe; `battenshed` is today the only producer emitting it.
- **D3 — Two new `HistoryEntry` fields, both `omitempty`.**
  `repeats` (int): the number of further identical calls folded into this entry, 0 and omitted when none.
  `last_at` (RFC 3339 UTC, same format as `at`): the time of the most recent folded call, omitted when none.
  `at` keeps the time of the first call in the run of folded entries.
  Rationale: the entry stays an honest record of the wait's start, end and poll count; `omitempty` keeps every existing status file and every unfolded entry byte-compatible.
- **D4 — Only consecutive entries fold.**
  Any other entry between two exempt Stucks — a counted Stuck, a Done, an Awaiting, another producer's entry — ends the fold, and the next exempt Stuck appends a fresh entry.
  A pause/resume write appends no history entry, so it does not break a fold; that is accepted.
- **D5 — `episodeStuckCount` is unchanged.**
  It already skips exempt entries, so folding them changes no bounce count and no budget decision.
- **D6 — `activity.last` after a fold reads as if the entry had been appended.**
  A fold is a history change even though the slice length is unchanged, so `persist` must recompose `activity.last` (with the `routedTo` wording, "`<producer>` → bounced to `<routedTo>`") rather than carry the previous value forward as it does for non-appending pause/resume writes.
- **D7 — The approval-acted marker also records the child's history length at resume.**
  When `callAwaiting` resumes an approved child, it records `len(status.History)` of the child status it read in that Call.
  The marker holds the approval identity line unchanged, followed by a second line with that length in decimal.
  On a later Call that finds the child still `awaiting` with the same approval identity:
  - child history length equal to the recorded length: the child's driver has not re-stepped yet — budget-exempt wait whose reason says the approval was acted on, the resume was delivered, and the child's driver has not re-stepped yet;
    if this persists, a driver that has ended is replaced by `lyx loom start` in the task worktree, and a live one is inspected in its pane.
  - any other child history length: the child ran and halted at a hand-off again — the existing re-approve hint, unchanged.
  Rationale: a re-await always appends at least the child's new Awaiting verdict, so length growth is a clock-free, exact discriminator; D1 folds only onto an exempt Stuck, never an Awaiting entry, so folding never masks that growth.
  The reason never calls `lyx loom start` a re-delivery: the resume removed the park marker, so start refuses a live driver as not parked.
- **D8 — A marker without a recorded length behaves as today.**
  A marker in the old one-field layout (written by a binary from before this change) matches on identity alone and yields the existing re-approve hint.
  Rationale: the marker lives in `.lyx` scratch and is per-run; no migration is worth building.
- **D9 — Marker comparison is on the approval identity alone.**
  The recorded length is carried beside the identity, never part of it, so a new approval (different `approved_at`/`head_sha`) still triggers a resume regardless of the recorded length.
- **D10 — `history_length` stays frozen during a folded wait, and `ly-drive` is unchanged.**
  A fold does not advance `history_length`.
  Batten wires no interrupt policy, so its `interrupt_policy` is `""`, and `ly-drive` hands back an interrupted `Run-Shed` step whose status read is unchanged instead of re-invoking it.
  A repair during a wait also no longer has its repair count reset by the successful polls around it.
  Rationale: a wait is not progress, and handing back is the conservative direction for an interrupted step.

## Constraints

- Shed Producer-Seam Invariant: `internal/shedengine` keeps importing only stdlib, `state`, `lock`.
- Transient Stop Invariant: a budget-exempt wait is a Stuck verdict and never carries the transient mark; unchanged.
- Told-Geometry Invariant: `battenshed` derives no path; the marker path stays `approvalActedFile(scratchDir, name)`.
- The single-persist-per-step crash-safety property in `Shed.Step` holds: the fold is computed in memory on a copy of the history read at step 1 and committed by the same one `persist` call; the read slice is never mutated in place.
- Status-file decoding: `ReadJSONStrict` rejects unknown fields, so both new fields are declared on `HistoryEntry` itself.
- Durable-vs-Ephemeral State Invariant: the approval-acted marker stays under the run's `.lyx` scratch directory.
- Docs land in the same commit (`docs/overview.md`); no new cross-cutting invariant, so `CONSTRAINTS.md` is untouched.
- Markdown edits use semantic line breaks.

## Auto-mode assumptions

- Folding is limited to budget-exempt Stucks; the task names awaiting waits, and counted Stucks are already bounded.
- Old history in an already-running bloated status file is left in place; folding applies from the next exempt Stuck on.
- A fold spanning a pause/resume is acceptable, since history never recorded pauses.
- The discriminator for the post-resume hint is the child's history length, not a timestamp comparison.
- A resume the child never picks up keeps reporting the "not re-stepped yet" reason indefinitely; the reason names the remedies (inspect a live driver's pane, `lyx loom start` for an ended one) rather than adding a timeout.
- A forward-compatibility gap (an older `lyx` binary strict-decoding a status file carrying `repeats`/`last_at`) is accepted, matching the precedent of `budget_exempt` and `transient`.

## Open risks

- **Mixed binaries.**
  A status file written by the new binary fails strict decode under an older `lyx` (for example production `lyx` reading a dev-build run) once it carries `repeats` or `last_at`.
  A two-line approval-acted marker fails an older binary's whole-file compare, so that binary attempts the resume again: it is refused as not parked while the resume is pending, and at worst resumes the child once more on the already-acted approval at its next hand-off.
  Accepted; in practice the outer status file already carries `repeats` by then, so the older binary fails strict decode first.

## Acceptance criteria

`internal/shedengine`:

- Consecutive budget-exempt Stucks from the same producer with equal `output` and `gate_attempts` leave history length unchanged after the first; the folded entry keeps its original `at`, `repeats` equals the number of folded calls, and `last_at` is the latest call's time.
- A counted Stuck following an exempt Stuck appends; an exempt Stuck following a counted Stuck appends; an exempt Stuck following an entry by a different producer appends; an exempt Stuck whose `output` or `gate_attempts` differs from the last entry appends.
- Fold, then a Done or Awaiting, then another exempt Stuck: the last one appends a fresh entry with no `repeats`/`last_at`.
- `episodeStuckCount` over a history with folded entries returns the same count as over the unfolded equivalent.
- After a fold, `activity.last` equals "`<producer>` → bounced to `<routedTo>`".
- A `HistoryEntry` with zero `repeats` and empty `last_at` marshals without either key; a status file written without them decodes under `ReadJSONStrict`.
- The step's single persist carries the fold, and the history slice read at step 1 is not mutated.

`internal/battenshed`:

- Driven through a real `shedengine.Shed` (the existing `innerrun_awaiting_test.go` harness), many awaiting polls with no approval leave the outer status history at one `Run-Shed` exempt-Stuck entry with a growing `repeats`, and the run is never blocked by the running budget.
- After a successful approved resume, a next Call that finds the child still `awaiting` with an unchanged history length returns a budget-exempt Stuck whose reason says the child's driver has not re-stepped yet and names both remedies from D7, and does not contain the re-approve hint; no second spawn happens.
- The approval-acted marker written on resume holds the unchanged identity line followed by the recorded length.
- A next Call that finds the child `awaiting` with a longer history and the same approval identity returns the existing re-approve hint, and does not spawn.
- A new approval identity resumes the child regardless of the recorded length.
- An old-layout marker (identity only) with the child awaiting yields the re-approve hint and no spawn.
- The `ErrChildNotParked` path still records no marker and retries on the next poll.

Docs:

- `docs/overview.md`'s batten `status` paragraph no longer claims one entry per awaiting or driver-exit poll, and describes the fold.

## Notes for the plan writer

- History append: `appendHistory` in `internal/shedengine/run.go`, used by every verdict arm; the fold belongs there or immediately beside it, so every arm that appends an exempt Stuck gets it.
- `callAwaiting`, `approvalActedFile`, `approvalIdentity` and the `Call` doc comment's disposition list are in `internal/battenshed/innerrun.go`; `Call` already holds the child `status` it passes to `callAwaiting`.
- `internal/battenshed/innerrun_awaiting_test.go` already builds a `shedengine.Shed` around the producer for budget tests.
- `internal/shedengine/run.go`'s `persist` doc comment and `internal/shedengine/status.go`'s `HistoryEntry` doc need the fold described.
