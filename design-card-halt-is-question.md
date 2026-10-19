# A halt a plan card asks for reaches the parent as a question, not as a batch failure

`card-halt-is-question` · Note · enhancement, webster, shed, loom

## Seen (2026-10-19, run `ubuntu-2404-support`)

Card 2 (`toolfloor-package`) told its implementer to halt, with a note, on any audited git or tmux feature above the 3.4 floor besides two known forms.
The fork found a third (tmux `allow-set-title`, 3.5), halted as the card said, and the batch was reported `FAILED`.
Webster then ran `recover-batch`, whose agent asked the orch the card's question directly, but ended before the answer came and refused with `batch_failed`, since the card still said halt.
The Go driver then spawned the tackler, which handed back because it could not find the decision; the next step was the arbiter.
So a halt the plan asked for travelled as a failure through three agent sessions, and the orch's answer (a parent decision in the decision record) reached none of them.

## Idea

A halt a card asks for is a question, not a failure.
It goes straight to the parent as a question: no `recover-batch`, no tackler, no arbiter.
Webster waits for the answer, and the next attempt at the batch gets the recorded decision, so a card whose halt condition the decision settles is carried out instead of halting again.
