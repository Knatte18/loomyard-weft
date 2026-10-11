# One seats Discussion producer: advisor count and lens forks as config axes

`seats-twill-combined` · Note · enhancement, loom, stencil, config

## Seen

`loom.yaml` `discussion_producer` takes one of `single`, `seats` and `twill`, so a run gets either a chair with advisors or a writer that forks itself into lens reviews, never both.
The chair stencil (`contracts/stencils/loom/loom-template-discussion-chair.md`) carries no twill directive.
Seats already runs on MultiLLMProducer (`internal/shedrecipe/entries_discussionseats.go`, `internal/shedadapters/multillm.go`), and `discussion_advisors` already runs the chair alone when the list is empty.

## Idea (operator, 2026-10-11)

One seats Discussion producer on MultiLLMProducer covers single, seats and twill, set by two config axes:

1. How many advisors the chair has, likely kept to one.
   - None: the chair decides each question itself, as the single writer does today.
   - One or more: the chair sends every question to the advisor as an open question (note `chair-blind-questions`); the advisor researches and answers with its own reasoning; the chair has already formed its own view and takes the advisor's answer as reflection, but the chair makes every call.
2. Whether the chair then forks itself for lens reviews of its draft, and how many forks with which lenses (today's `twill-discussion` fan), as twill does.
   The chair forks, never the advisor: the forks share the chair's prompt cache, and the advisor already brings the outside view.
   Advisor forks are possible later but not the way to go yet (operator, 2026-10-11).

With both axes at zero this is today's single writer, so `discussion_producer` could go.

## Open

- Bench the combination against the arms of the twill bench and the chair-blind-questions bench (`.scratch/twill-bench/results.md` holds the earlier arms while it lasts).
