# Discussion chair asks advisors open questions without its options or pick

`chair-blind-questions` · Note · enhancement, loom, stencil

## Seen

`contracts/stencils/loom/loom-template-discussion-chair.md` tells the chair to send each question with its recommended answer and its alternatives to every advisor.
The advisor then reads the chair's pick before it studies the question, so its answer is anchored on the chair's options and lean, and it rarely finds an option the chair did not list.

## Idea (operator, 2026-10-11)

- The chair still forms each question with its options and its own recommended answer, but keeps them to itself.
- It sends the advisors the bare question only, so each advisor explores and answers on its own.
- The chair then weighs the advisors' answers against its own pick; an answer outside its options counts as a new option.
- Questions must therefore be open, never leading: a question must not name or hint at the chair's preferred answer.

## Open

- Whether the ledger records the chair's private pick before the advisors answer, so a review can see where an advisor changed the outcome.
- Whether the advisor stencil (`loom-template-discussion-advisor.md`) needs a line saying it gets no options and should name the alternatives it sees.
