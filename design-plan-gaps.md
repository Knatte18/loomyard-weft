# Plan gaps: inserting a card, unreachable prescribed refusals

`plan-gaps` · Task · enhancement, loom, burler, stencil · running · Plan-Bouncer

Two gaps in how plans are written and reviewed, merged into one task (operator, 2026-10-11).

## Inserting or removing a card (was `plan-card-insert`, issue #508)

Plan cards are numbered contiguously and each card file name is tied to its number.
A Plan-Review fix that needed one new card ahead of card 8 therefore renumbered cards 8 to 19, twelve files.

The fixer rules allow file changes only through Edit or Write and name no way to rename or remove a file.
The fixer rewrote each card under its new name and removed the old files with `rm`, which the rules neither allow nor forbid.

This is abnormal: a routine plan edit forces a fixer to guess at an unstated rule and do a bulk, error-prone rewrite.

What to do, either of:

- add a plan verb that inserts a card and renumbers the rest;
- state a rule for removing a file inside the fixer's write surface.

Provenance: task `driver-context`; note `burler-plan-r1.md`.

## Unreachable prescribed refusals (was `plan-refusal-reachability`, issue #516)

A plan card prescribed a refusal branch (a zero-batch refusal after an accepted auto-rebaseline) with a fixed signature.
Review of the implementation flagged that no test reaches the refusal; the fixing agent then found that no plan can reach it, because the plan parser refuses a cardless plan before `Run` forms any batch.
The branch was kept and its text pinned with a golden test of the error constructor, which conflicts with the rule against code for impossible states.

This is abnormal because the plan passed its Plan-Bouncer and Plan-Burler rounds with a prescribed refusal that is unreachable by construction, and the contradiction surfaced only after the code was written.

What to do: have plan review (or the plan-time checks) verify that every refusal a card prescribes is reachable from some input that passes upstream validation, or require the card to say how a test reaches it.

Provenance: task `webster-replan-state`, from note `burler-webster-r1.md`.
