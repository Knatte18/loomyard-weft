# A question research can settle is settled in Discussion, never deferred to Plan or an implementer

`research-in-discussion` · Note · enhancement, loom, stencil, burler

## Seen (2026-10-19, run `ubuntu-2404-support`)

The task's goal included researching which git and tmux features lyx uses above the Ubuntu 24.04 floor (git 2.43, tmux 3.4).
Discussion and Plan settled two above-floor forms without auditing the full list against tmux's `CHANGES`.
Plan card 2 (`toolfloor-package`) was given the audit itself, with a rule to halt on any third above-floor form.
The implementer fork did the research in about two minutes (tmux `CHANGES` plus `tmux.1` at tags 3.4 and 3.5), found `allow-set-title` (3.5), and halted.
The halt cost a failed batch, a recover-batch, a tackler, an orch decision and a `goto Plan-Write` with a fresh Plan review.

## Operator decision (2026-10-19)

A question research can settle is settled in Discussion, never deferred to Plan or to an implementer with a halt rule.

## Idea

- The Discussion writer stencil says so: a factual question the design rests on (versions, upstream behaviour, what the code does today) is researched and its answer, with the evidence, recorded in the discussion.
- The Discussion reviewer finds a deferred research question: a decision or criterion that leaves a fact to be found out later, or a halt-on-discovery rule handed downstream, is a finding.
- The Plan reviewer treats a card that carries a research step whose answer the design depends on as a finding.
