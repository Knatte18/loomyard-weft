# Bench fork0 and the strict critical fixer on twill Discussion-Write

`twill-discussion-fork0-bench` · Note · audit, loom, burler, stencil

## Ask (operator, 2026-10-15)

Bench fork0 and the strict critical fixer on twill's Discussion-Write, as the Plan-Write benches did on 2026-10-14.
The operator designed twill mainly for Discussion, to give a far more solid first draft, and holds that a watertight Discussion saves the most later in a run.
On Plan-Write the gain came above all from tightening the prompt of the part that makes the changes the lens forks reported: the agent doing it must doubt every finding.

## Today

- Discussion-Write runs twill (`discussion_producer: twill`) with the `twill-discussion` lens fan in `internal/burlerengine/template.yaml`, unchanged since twill landed (`50cf6a425`) and benched once (`docs/research/twill-discussion-bench.md`).
- The task `twill-fork0` builds fork0 for Plan-Write only; its Discussion auto-picked fork0 out of Discussion-Write because no bench measured it there, and applies the critical judge to Discussion-Write without a bench.

## Bench

- Fresh agent, the same method as `docs/research/twill-fixer-bench.md` and `docs/research/twill-fork0-bench.md`: references with known Discussion-Review findings as the key, arms spliced into the Discussion-Write prompt by a wrapper, runs stopped after Discussion-Write, one grader in one pass.
- Arms:
  - a: twill as it is (writer writes, then lens forks, then the writer fixes).
  - c: a with the strict critical fixer (doubt every finding, two-sided weighing).
  - f: fork0 (the master explores and interviews, fork0 writes the draft, the master spawns the blind lens forks and fixes as the strict critical fixer).
- Measure: findings real/noise, wrong dispositions, key items repeated in the final discussion, cost and time per arm.
- Question for arm f: the master holds the interview context the fork does not, so whether fork0 loses anything at Discussion is the point to grade.

## Timing

Open: bench on a prototype wrapper now, as the Plan benches did, or after `twill-fork0` lands, so the bench runs on its built critical judge.
