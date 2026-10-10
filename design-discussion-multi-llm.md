# Discussion-Write on MultiLLMProducer: a chair writes, advisors answer

`discussion-multi-llm` · Task · feature, loom, stencil, shed · running · Discussion-Bouncer

Depends on: [`multi-llm-producer`](design-multi-llm-producer.md), [`gate-test-scope`](design-gate-test-scope.md)

## Goal

Design and build a Discussion-Write that runs on `MultiLLMProducer`: the writer is the `chair`, with one or two advisors, so the Discussion is designed by agents working together (operator, 2026-10-08: a good Discussion protocol is worth the tokens).

## From `discussion-advisor`

- **Ask mode**: the chair asks its design questions as they arise, and an advisor replies; answers to design questions, not a review.
- An advisor is spawned in parallel with the chair with its own stencil: it studies the task, the board and the code while the chair drafts, so it is ready when the questions come.
- The seats are live sessions that keep their context, so this is a real dialogue, and none of it lands in the parent's context.
  Agents designing together beat one agent asking and answering itself.
- `ask` and the parent review are independent switches: advisors answering the chair and the parent reviewing the draft can both run on one task.

## Open for Discussion

- How many advisors, and what each one is for: the same brief twice, or distinct roles such as one that reads the code and one that challenges the design.
- When the chair ends the step, and what an advisor's output in the decision record looks like, if any.
- Model per seat, and the config keys for them.
- Whether the operator's interview turns still go to the chair alone.

## Models (operator, 2026-10-08)

Discussion runs on `fable[high]` as a single agent from 2026-10-08.
Once this lands, the operator expects to step down to `opus[high]` for the chair with a `sonnet[high]` advisor.
