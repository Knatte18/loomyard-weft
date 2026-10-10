---
round: 1
focus:
  - "D2, D4, D5, D8, D9: for each new edge (detached loop, watchdog kill, `failed` write by the loop, loop-made transient re-step, driver fork), does the Decision state what it can now skip or let through and what bounds it? Check the bound is concrete (D2's pid-file liveness and unwatched-envelope delivery, D5's write on a still-`running` file only, D9's fork edit limits)."
  - "D3 and D5 against `PATTERN-failure-stop-visible` and the acceptance criteria: a `--clear` that removes the bare pause request, a condition cleared in the persist that records `paused`, and a loop-written `failed` over a status file another holder may own. Does each guard change name its bound, and does an acceptance criterion reach it?"
  - "D1, D4, D6 consistency: the stop-cause set, the `loop` key set and the step-kind vocabulary (`interrupted` as sixth, `bootstrap` from D6) agree across Decisions, Constraints and Acceptance criteria. Is `bootstrap` already one of the five kinds, or does the 'five to six' claim in D4 miscount?"
  - "Relocation: the Rationale lines, the 'Added after Discussion' entry's run history, and any deliberation narrative in Decisions. Apply the completeness-before-leanness test first; does any of it carry a requirement the plan writer needs that is not stated elsewhere in `decision-record.md`?"
  - "Scope against Decisions: every Scope 'In' bullet has a Decision or Constraint that settles it, and no Decision introduces behavior absent from Scope (for example the PATTERN overview byte cap, D10's SIGQUIT routing)."
  - "Premises in Auto-mode assumptions: which of them, if false, would make a Decision wrong rather than merely costlier, and does the Decision or Open risks say what happens then?"
---

Seed focus only; no verdict is pre-stated.
Items on the rubric's Do not flag list (rejected alternatives in the record, optional plan-writer notes, call-site enumeration) were left out on purpose.
Exact signatures and file:line facts are not asked for, per the writer stencil.
