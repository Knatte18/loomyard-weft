# Seats replaces SingleLLMProducer: every LLM producer is a MultiLLMProducer, zero advisors by default

`seats-replaces-singlellm` · Note · enhancement, loom

## Seen

Two general LLM producers live side by side in `internal/shedadapters`: `SingleLLMProducer` (`singlellm.go`, with a gated variant `NewSingleLLMProducerGated`) and `MultiLLMProducer` (`multillm.go`, on `internal/seatengine`).
Writers such as Discussion-Write, Plan-Write and Darn-Write run on `SingleLLMProducer`; seats Discussion (`internal/shedrecipe/entries_discussionseats.go`) and, since `7882c2450`, the Burler round run on the seats engine.
`loom.yaml` `discussion_producer` picks one of `single`, `seats` and `twill`, and `discussion_advisors` already runs the chair alone when the list is empty.

## Idea (operator, 2026-10-11)

Seats is the general producer; the single-LLM producer is the special case.
Every `SingleLLMProducer` user becomes a `MultiLLMProducer` with zero advisors, and `SingleLLMProducer` goes.
The goal is less code: one LLM producer instead of two.

For Discussion this gives two config axes in place of `discussion_producer`:

1. How many advisors the chair has. None: the chair decides each question itself, as the single writer does today.
2. Whether the chair forks itself for lens reviews of its draft, and with which fan (today's `twill-discussion`), as twill does.
   The chair forks, never an advisor: the forks share the chair's prompt cache.

Zero advisors with the fan on is today's twill, the hub's current setting.

## Required

- The seats producer carries the same gates as the gated single-LLM producer (operator, 2026-10-11): a role that runs gated today stays gated after the move.
- Seats are named by the recipe that uses them (operator, 2026-10-11): "chair" and "advisor 1" mean nothing for Burler, whose seats are the fixer and the reviewer, and a table with one seat has no chair, only its role (writer, planner and so on).
  The names reach role names, strand names, prompts and logs; "chair" and "advisor" stay as the engine's internal terms at most.
  `burler-split-budget` already runs Burler as a reviewer row and a fixer row, each a zero-advisor table, under distinct role names.

## Bench basis

Round 4 of the twill bench (2026-10-11) ran the Discussion combination (blind advisor plus chair lens forks): 2.5 / 2.0 repeated / partly repeated reference defects at $7.52, against twill alone at 1.5 / 2.0 for $4.75.
The advisor axis added nothing there, so the merged producer keeps it available, not on by default.

## Open

Not yet mapped against the code:
- what the single-LLM producer does that the seats producer lacks besides gates: `prepareFreshSpawn`, interactive runs (`discussion_interactive`), spec sourcing per role;
- the overhead the seat table and its hold deferral add to a chair with no advisors.
