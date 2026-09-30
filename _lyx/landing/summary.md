# Fold budget-exempt waits in status history

A batten run's `status.json` no longer grows without bound while `Run-Shed` waits on a human hand-off.
`Run-Shed` also stops telling the operator to re-approve on the poll right after it resumed an approved child.

## Why

Every 30 s poll of an awaiting child returns a budget-exempt Stuck, and `shedengine` appended one `history[]` entry per poll.
`max_bounces` caps counted Stucks but not exempt ones, so a multi-day approval wait wrote tens of thousands of near-identical entries.
Separately, after `callAwaiting` resumes an approved child, the child stays `awaiting` until its driver re-steps.
A poll in that window matched the approval-acted marker and reported "already acted on and the child is awaiting again; re-run `lyx loom approve`", which was wrong.

## What changes

`internal/shedengine`:

- A budget-exempt Stuck folds into the immediately preceding entry when that is also a budget-exempt Stuck with the same `producer`, `output` and `gate_attempts`.
  The entry keeps its first `at`, and gains `repeats` (further calls folded in) and `last_at` (time of the latest one).
  Both fields are `omitempty`, so existing status files and unfolded entries are byte-compatible.
- The fold is engine-generic and keyed on `BudgetExempt`; there is no producer flag.
  It covers every exempt wait, including `Run-Shed`'s driver-exit grace, not only the approval wait.
- Only consecutive entries fold.
  Any other entry in between (counted Stuck, Done, Awaiting, another producer) ends the fold.
- Counted Stucks still append one entry per call, bounded by `max_bounces`.
- `persist` recomposes `activity.last` after a fold, so it reads as if the entry had been appended.

`internal/battenshed`:

- The approval-acted marker now holds the approval identity line followed by the child's history length at the resume.
- A later call that finds the child still `awaiting` under the same approval compares the child's history length to the recorded one.
  Equal means the resume was delivered and the driver has not re-stepped yet; the exempt-Stuck reason says so and names the remedies.
  Any other length means the child halted at a hand-off again, and the existing re-approve hint stays.
- The "not re-stepped" reason points at `lyx loom start` only for an ended driver and at the pane for a live one, because the resume removed the park marker and `start` refuses a live driver as not parked.

Docs: `docs/overview.md`'s batten `status` paragraph and the `battencli` history-cap comment no longer claim one entry per awaiting poll and describe the fold.

## Design decisions

- **Fold, not cap.**
  Exempt Stucks are unbudgeted waits, so the fold keeps the file O(1) per wait episode and loses only per-poll timestamps.
- **History length as the discriminator.**
  A re-await always appends at least the child's new Awaiting entry, and the fold never folds onto an Awaiting entry, so length growth is an exact, clock-free signal.
- **Marker compared on identity alone.**
  The recorded length sits beside the identity, so a new approval still triggers a resume whatever length was recorded.
  An old one-line marker matches on identity and yields the re-approve hint; no migration.
- **`episodeStuckCount` and `history_length` behaviour.**
  `episodeStuckCount` already skips exempt entries, so no budget decision changes.
  A fold does not advance `history_length`, so `ly-drive` hands back an interrupted `Run-Shed` step with an unchanged status read instead of re-invoking it.
- **A pause/resume write appends no history entry**, so it does not break a fold.

## Check before landing

- **Mixed binaries.**
  An older `lyx` strict-decoding a status file that carries `repeats` or `last_at` fails, for example production `lyx` reading a dev-build run.
  This matches the `budget_exempt` and `transient` precedent and is accepted.
- **No retroactive compaction.**
  History already written by an in-flight run stays as is; folding applies from its next exempt Stuck.
- **Not run by hand:** a real multi-hour approval wait against a live batten run, to confirm `status.json` stays small and the post-resume reason appears.

No issue is linked: the board entry cites PR #304 as where this was found and names no issue number, so there is no `Closes` line.
