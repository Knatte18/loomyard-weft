# Fixer report — discussion round 3

## Changes

- F1 (MEDIUM, fixed as proposed): `_lyx/discussion/decision-record.md` D3 now states, after the sentence naming the two replaced relaunch paragraphs, that the re-prompt rule also replaces the directive's sentence "A gate re-prompt, a parent-review reject or a later turn of the same session never spawns forks again." at every host, Discussion-Write included, and that a reed resume of the same conversation is governed by the resumed paths or, at a host with no writer fork, by the two relaunch paragraphs, never by that sentence.
  Acceptance criteria: the Plan-Write item gains a sub-item that the rendered prompt carries no copy of that sentence, and the Discussion-Write item says the lead prompt carries neither a fork0 part nor that sentence.
  Premise confirmed at `contracts/stencils/twill/twill-directive.md:20-21`.
- F2 (LOW, fixed as proposed): the Acceptance criteria's D3 failure-path sub-item now names the re-prompted master's whole rule: no fork0, reads the plan per D2 step 4 first if it has not, runs the lens review once if it has not made its lens spawn call, and spawns no lens forks if it has.

## Disputed

None.

## Deferred

None.
