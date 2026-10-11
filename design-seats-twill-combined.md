# One Discussion producer: seats with optional advisors and chair lens forks

`seats-twill-combined` · Note · enhancement, loom

## Seen

`loom.yaml` `discussion_producer` takes one of `single`, `seats` and `twill`, each a separate code path.
Seats runs on MultiLLMProducer (`internal/shedrecipe/entries_discussionseats.go`, `internal/shedadapters/multillm.go`), and `discussion_advisors` already runs the chair alone when the list is empty.
`single` and `twill` run on the single-LLM writer producer instead.

## Idea (operator, 2026-10-11)

Seats is the general producer; the single-LLM producer is the special case.
Run every Discussion through seats, set by two config axes, and drop the single-LLM path and `discussion_producer`:

1. How many advisors the chair has. None: the chair decides each question itself, as the single writer does today.
2. Whether the chair forks itself for lens reviews of its draft, and with which fan (today's `twill-discussion`), as twill does.
   The chair forks, never an advisor: the forks share the chair's prompt cache.

Both axes at zero is today's single writer; zero advisors with the fan on is today's twill, the hub's current setting.
The goal is less code: one Discussion producer instead of two.

## Bench basis

Round 4 of the twill bench (2026-10-11) ran the combination (blind advisor plus chair lens forks): 2.5 / 2.0 repeated / partly repeated reference defects at $7.52, against twill alone at 1.5 / 2.0 for $4.75.
The advisor axis added nothing there, so the merged producer keeps it for later use, not as a default.

## Open

- Whether other users of the single-LLM writer producer (Plan, darn) can move to seats the same way, or keep it.
