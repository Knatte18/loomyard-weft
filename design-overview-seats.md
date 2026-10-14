# Overview window tidy, seat names, Seats replaces SingleLLMProducer

`overview-seats` · Task · enhancement, reed, shed, loom, burler · running · Plan-Burler-Fix

This is one task because the three notes all touch the seat/producer layer and how runs are shown in the overview window.
Each note's content is kept under its own heading below.

## Overview window: one combined log and checklist, no stuck for a bounce

### Seen (2026-10-14, `tender`'s overview window)

- The window shows a log (one line per attempt, with time, outcome and file) and below it a checklist (one line per recipe row, `[x]`, `[>]`, `[ ]`, with total time and attempt count).
  A finished row so stands twice, and the operator read it as a duplicate.
- Plan-Bouncer showed `22h18m`: the pause and the overnight hibernate count as the row's time, though the row itself took 0 minutes.
- Bouncer and Burler rows that bounce to the next round read `stuck`.

### Wanted (operator, 2026-10-14)

- No `stuck` for a bounce: the state machine is not stuck, a bounce is part of the normal flow; `stuck` would mean something sits fast.
  Rename the outcome where it is shown (and consider the name in the shed recipe's `OnStuck` too), for instance `bounced`.
- Make it clear which part is the log and which the checklist, or drop finished rows from the checklist since the log holds them.

### Idea (operator, 2026-10-14)

One combined view instead of two parts: the checklist in recipe order, and under each row its log lines (time, outcome, file) fill in as the row runs.
Rows still to run stay as bare `[ ]` lines, so the view shows both what happened and what is left.

### Seen as well (operator, 2026-10-14)

- The log groups lines by recipe row, so Discussion-Bouncer's seven lines stand before all Burler-Review and Burler-Fix lines, though the three rows took turns, round by round (bouncer, review, fix, bouncer, ...).
  The order the run took is lost, and the log is hard to follow.
- The arbiter is not in the log at all: round 3's circling escalation and its `continue` decision do not show.

A view that fits both: within a review segment, lines grouped by round (round 1: bouncer, review, fix; round 2: ...), with the arbiter's spawn and decision shown in the round it decided.
This refines the combined view below: a segment's rows fill in round by round, not row by row.

### To consider

- Paused time (`lyx pause`, hibernate) should not count as a row's time.

## Seat names: none for a lone seat, operator-chosen names for several, no chair

### Seen (2026-10-14)

Discussion and every Burler round run through the seat engine (`internal/seatengine`), even with one agent.
A seat's name is fixed: `seatengine.RoleChair` (`chair`) for the leading seat and `advisor-<n>` for the others (`Seat.Name` in `internal/seatengine/table.go`).
The name goes into the strand and role names the operator sees, such as `burler-review-chair` and `burler-fix-chair` (`internal/burlerengine/doc.go`), even in a round with one seat.
A recipe's MultiLLM row has a `name` key per seat, but the engine expects `chair` and `advisor-<n>`.

### Wanted (operator, 2026-10-14)

- A lone seat carries no seat name: the role reads `burler-review`, not `burler-review-chair`.
- With two or more seats, the operator names each seat in the recipe or config, and that name is what the windows, strands and overview show.
- `chair` is not a name to show the operator.

### To consider (operator, 2026-10-14)

A name that shows which round a session belongs to, since `burler-review-chair` does not say which round it is.
The operator notes this may matter less now that the overview window shows the rounds.

### To check

- Where `chair` and `advisor-<n>` are load-bearing: stencils (`seat-directive-chair`, the `advisor_names` marker), strand names, reed window names, and any lookup by role.
- Whether the leading seat is marked by its place in the table or by a flag, once the name is free.

## Seats replaces SingleLLMProducer: every LLM producer is a MultiLLMProducer, zero advisors by default

### Seen

Two general LLM producers live side by side in `internal/shedadapters`: `SingleLLMProducer` (`singlellm.go`, with a gated variant `NewSingleLLMProducerGated`) and `MultiLLMProducer` (`multillm.go`, on `internal/seatengine`).
Writers such as Discussion-Write, Plan-Write and Darn-Write run on `SingleLLMProducer`; seats Discussion (`internal/shedrecipe/entries_discussionseats.go`) and, since `7882c2450`, the Burler round run on the seats engine.
`loom.yaml` `discussion_producer` picks one of `single`, `seats` and `twill`, and `discussion_advisors` already runs the chair alone when the list is empty.

### Idea (operator, 2026-10-11)

Seats is the general producer; the single-LLM producer is the special case.
Every `SingleLLMProducer` user becomes a `MultiLLMProducer` with zero advisors, and `SingleLLMProducer` goes.
The goal is less code: one LLM producer instead of two.

For Discussion this gives two config axes in place of `discussion_producer`:

1. How many advisors the chair has. None: the chair decides each question itself, as the single writer does today.
2. Whether the chair forks itself for lens reviews of its draft, and with which fan (today's `twill-discussion`), as twill does.
   The chair forks, never an advisor: the forks share the chair's prompt cache.

Zero advisors with the fan on is today's twill, the hub's current setting.

### Retiring the single producer (operator, 2026-10-10; moved from `discussion-multi-llm-bench`)

The seats producer with an empty `discussion_advisors` list covers the single writer's job: one chair prompt for every advisor count says "use your advisors if you have any; with none, decide each question yourself as you judge best", with no conditional rendering.
Once that is tested, the single producer is parked, since seats with N=0 covers it; the bench should include an N=0 run against solo to show they match.

### Required

- The seats producer carries the same gates as the gated single-LLM producer (operator, 2026-10-11): a role that runs gated today stays gated after the move.
- Seats are named by the recipe that uses them (operator, 2026-10-11): "chair" and "advisor 1" mean nothing for Burler, whose seats are the fixer and the reviewer, and a table with one seat has no chair, only its role (writer, planner and so on).
  The names reach role names, strand names, prompts and logs; "chair" and "advisor" stay as the engine's internal terms at most.
  `burler-split-budget`'s design (in flight) runs Burler as a reviewer row and a fixer row, each a zero-advisor table, under distinct role names.

### Bench basis

Round 4 of the twill bench (2026-10-11) ran the Discussion combination (blind advisor plus chair lens forks): 2.5 / 2.0 repeated / partly repeated reference defects at $7.52, against twill alone at 1.5 / 2.0 for $4.75.
The advisor axis added nothing there, so the merged producer keeps it available, not on by default.

### Open

Not yet mapped against the code:
- what the single-LLM producer does that the seats producer lacks besides gates: `prepareFreshSpawn`, interactive runs (`discussion_interactive`), spec sourcing per role;
- the overhead the seat table and its hold deferral add to a chair with no advisors.
