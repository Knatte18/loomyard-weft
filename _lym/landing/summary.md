# Discussion writers start their exploration with explore subagents

The Discussion-Write stencil, `contracts/stencils/loom/loom-template-discussion.md`, now asks the writer to open Step 2 with one batch of explore subagents before it reads code in its own context.
Once the board entry is read, the writer lists the questions the exploration raises and sends the independent ones together to fresh `general-purpose` subagents on `subagent_model`, in one message.
After the batch returns, it reads in its own session only the code it needs to see, batching each turn's reads and searches.

## Why

The writer-lookups measurement (`docs/research/writer-lookups-measure-2026-11-04/report.md`) found that the Plan writers, whose stencil asks for an early lookup batch, all sent one, while 2 of 3 full Discussion sessions, whose stencil left delegation optional, sent none and explored in serial chains of 15 and 20 turns.
The larger cost was carry: what an exploration turn reads stays in context for every later turn.
A subagent returns only what bears on its question, so starting with one shrinks the session's context for its whole length, which the stencil now states as the reason.

## Design decisions

- The old "whether to is your call" sentence is gone: the batch is the default start, not an option.
- The bounded-exploration fence above the paragraph still binds, and each subagent prompt is told to keep to it, so a subagent gathers no plan-level detail either.
- The fork exclusion stays, since a fork inherits the session's whole context and saves no carry.
- No Go code, marker or config key changes: `subagent_model` was already a required marker of this stencil.

## Review checks

- Step 2 of the rendered Discussion-Write prompt asks for the subagent batch before any code read in the session, with `model` filled from `loom.yaml`'s `subagent_model`.
- The follow-up measurement is filed as board note `writer-lookups-remeasure`.
