# MultiLLMProducer: one producer runs a chair and its advisors as seats

`multi-llm-producer` · Task · feature, shed, shuttle · running · Webster-Bouncer

## Goal

`MultiLLMProducer` in `internal/shedadapters`, beside `SingleLLMProducer`: one producer runs N agents in one step, each in a strand of its own.
It is built standalone and proven by its own tests and a test recipe: no recipe row, Burler or Discussion uses it in this task (operator, 2026-10-08).

## Terms (operator, 2026-10-08)

- A **seat** is one agent in a `MultiLLMProducer`.
- Exactly one seat is the **`chair`**: its turn end, outputs and gates decide the step's outcome, as a single agent's do today, and a message sent to the producer goes to the chair.
- Every other seat is an **advisor**, named `advisor-1`, `advisor-2` and so on, in every `MultiLLMProducer`.
- Strands are named after the seat, such as `ly:<slug>:discussion-chair` and `ly:<slug>:discussion-advisor-1`.
- Not "master": that is Webster's `Master`.

## Seats: one table says what goes where

Burler's round today wires six stencils (two orchestrator templates, three step files, a focus directive) plus per-segment rubrics and the Bouncer's judge stencils through Go code spread over several packages.
A `MultiLLMProducer` declares its seats in one place (the recipe row or one Go table):

- the seat's name (`chair` or `advisor-<n>`);
- exactly one stencil per seat, the only prompt that agent gets; shared building blocks are included from it by name;
- a model key in config;
- inputs (files and markers the seat is given) and outputs (files it writes); an advisor's output is the chair's input.

A test pins that every seat's stencil markers are filled (`stencil.Fill` already refuses an empty marker) and that every advisor output is the chair's input.

## Messaging

This first version carries its own internal message channel between seats; `creel` is not used and comes later on its own (operator, 2026-10-08).
The chair always steers: a message to the producer goes to the chair, and advisors reach the chair through the channel.
Whether advisors may message each other is decided in Discussion.

## Shape

Inside one producer, not parallel shed producers (operator, 2026-10-07): shed sees one step with one outcome, and the seats are invisible to it.
A single-agent producer is the case N = 1; whether `SingleLLMProducer` itself becomes that case is decided in Discussion, without changing any recipe row.

## Open for Discussion

- What an advisor's failure or hang does to the step.
- How the producer knows an advisor is done (the session-state model from `shuttle-state`).
- How the seats' strands are laid out until `reed-layout-shares` lands.

## Out

- Burler on `MultiLLMProducer` and Discussion-Write on it are their own tasks.
- `creel`, and the even layout (`reed-layout-shares`).
