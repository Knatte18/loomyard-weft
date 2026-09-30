# Support log — awaiting-status-history-growth

## Interview

Autonomous session: no operator answered; every decision below is a self-pick recorded in the question ledger.

1. **Task read.**
   Board task 29: each 30 s poll during an unbounded approval wait appends a history entry; collapse repeated identical wait entries or cap them.
   Related: right after a resume a poll can still see `awaiting` and logs a misleading re-approve hint.
2. **Where the growth comes from.**
   `battenshed`'s `innerRunProducer` returns a budget-exempt Stuck for the awaiting arms and the done-grace arm, and a counted Stuck for the running arm.
   `shedengine.Step`'s `appendHistory` appends one entry per verdict.
   Counted Stucks are bounded by `Run-Shed`'s `max_bounces: 1440`; exempt Stucks are unbounded.
   History entries carry no reason text, so consecutive exempt Stucks from `Run-Shed` differ only in `at`.
3. **Fold vs cap.**
   Folding consecutive identical exempt entries is O(1) per wait and loses only per-poll timestamps; a cap still grows to N and drops the start of the wait.
   Chose fold, with `repeats` and `last_at` so the entry keeps the wait's span and poll count.
4. **Where the fold lives.**
   History is `shedengine`-owned; producers cannot shape it.
   `BudgetExempt` already marks a wait, so keying the fold on it needs no new producer flag.
5. **Downstream consumers of history.**
   `episodeStuckCount` skips exempt entries, so budgets are unaffected.
   `history_length` is read by `ly-drive` (progress key for interrupted invocations, repair-cap key), by `lyx shed step`/`status` envelopes, and by loom's self-report/step-handoff logic.
   Loom never emits exempt Stucks, so loom-side consumers are unaffected; `ly-drive` driving a batten run sees no `history_length` change across a folded wait, recorded as an open risk.
   `persistTransient` recomposes `activity.last` only when the slice grew, so a fold must be recognised explicitly.
6. **Misleading re-approve hint.**
   `callAwaiting` resumes the child through `Spawn`; for a parked LLM driver the bootstrap types one resume line and returns while the child's status is still `awaiting`.
   The next poll matches the approval-acted marker and reports "already acted on … re-run `lyx loom approve`".
   The child's history length at resume time separates "not re-stepped yet" (length unchanged) from "awaiting again" (a new Awaiting verdict appended).
   Chose recording that length beside the identity in the existing marker.
7. **Testing.**
   `shedengine` unit tests for the fold rule and its boundaries, activity wording and JSON compatibility;
   `battenshed` tests through the existing real-`Shed` harness for the multi-poll fold and the post-resume dispositions.

## Rejected alternatives

- **Cap history globally (ring buffer).**
  Drops the durable record, breaks `episodeStuckCount`'s scan back to a Done boundary, and loom's coherence and anomaly checks read the whole history.
- **Cap only exempt entries at N per episode.**
  Still grows to N and discards the start of the wait; folding gives the same bound at size one with the start preserved.
- **Fold counted Stucks too, with a count-aware `episodeStuckCount`.**
  Counted Stucks are already bounded by `max_bounces`; widening the change to budget accounting is outside the task.
- **An opt-in producer flag (`OutputPointer.Collapse`) instead of keying on `BudgetExempt`.**
  Adds a second flag carrying the same meaning as the first.
- **Keep the running arm counted but have `battenshed` sleep internally until the approval arrives.**
  Contradicts the one-bounce-per-Call design that keeps pause requests read between rows and keeps each poll observable.
- **Return an Awaiting outcome from `Run-Shed` while the child awaits.**
  Halts the outer run, so the outer run would no longer resume the child on approval by itself.
- **Suppress the re-approve hint for a fixed grace window after a resume.**
  A timer guesses how long the child's driver takes to re-step; the history-length comparison is exact.
- **Compare the child's last history `at` against a recorded resume time.**
  Second-precision timestamps and a clock dependency, where the history length needs neither.
- **Retroactively compact an existing bloated history on the next persist.**
  Runs are per-task and short-lived relative to the change; no migration is worth the extra rewrite logic.

## Review rounds

_No rounds yet._

## Question ledger

- **Q1 — Fold or cap?** Resolved, auto-pick: fold consecutive identical budget-exempt Stucks (D1).
- **Q2 — Engine-generic or batten-specific?** Resolved, auto-pick: engine-generic, keyed on `BudgetExempt` (D2).
- **Q3 — Record the fold, or overwrite `at` silently?** Resolved, auto-pick: keep the first `at`, add `repeats` and `last_at`, both `omitempty` (D3).
- **Q4 — Fold counted Stucks too?** Resolved, auto-pick: no; out of scope, already bounded (Scope).
- **Q5 — Does a pause/resume break a fold?** Resolved, auto-pick: no; pauses append no history entry (D4).
- **Q6 — Compact existing bloated history?** Resolved, auto-pick: no (Scope, Auto-mode assumptions).
- **Q7 — How to tell "resume not picked up yet" from "awaiting again"?** Resolved, auto-pick: child history length recorded at resume (D7).
- **Q8 — Old marker layout?** Resolved, auto-pick: identity-only match, existing re-approve hint (D8).
- **Q9 — Timeout on a resume never picked up?** Resolved, auto-pick: no timeout; the reason names `lyx loom start` as the remedy.
- **Q10 — Is `history_length` freezing during a folded wait safe for `ly-drive`?** Open, carried to Open risks for the plan writer to confirm against batten's `interrupt_policy`.
- **Q11 — Forward-compat of the new fields under an older binary's strict decode?** Resolved, auto-pick: accepted, same precedent as `budget_exempt` and `transient`.
