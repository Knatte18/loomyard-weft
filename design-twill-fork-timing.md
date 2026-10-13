# Twill: spawn lens forks before or after writing

`twill-fork-timing` · Note · enhancement, loom, stencil

## Idea (operator, 2026-10-13)

Twill's writer spawns its lens forks before it writes its output, and the forks then wait for the ready marker and read the written files.
None of them has those files in its cache, so every fork pays a fresh cache write for them: once per fork.
Spawned after the writing, the forks would inherit the written files in their context as cache reads.
Test Discussion's twill first (operator, 2026-10-13).

## Seen

- The spawn point is one sentence per producer: `internal/loomengine/twill.go:91` for Discussion ("When the design is settled and before you write any output file"), `planTwillSpawnPoint` at `twill.go:66` for Plan.
- The forks wait on the ready marker and read nothing before it (`contracts/stencils/twill/twill-directive.md`, the fork boilerplate), so spawning early buys no parallel work; in the Plan bench they waited about five minutes each (`docs/research/twill-plan-bench.md:172`).
- Spawning before writing was never benched on its own: in `twill-plan` round 1 both writers spawned after writing by mistake, and round 2 changed the spawn point together with the rejection rule and the lenses (`docs/research/twill-plan-bench.md`, "Changes to the prototype").
- What spawning early may buy: a smaller inherited context per fork turn, and forks that do not see the writer's drafting, so they judge the written files less anchored to its reasoning.

## Bench result (2026-10-13)

Discussion-Write on the `webster-stability` reference, today's binary, two fresh runs per arm (report `docs/research/twill-fork-timing-bench.md`):

- Spawning after writing cost $4.44 per run against $5.43 (18 % less), all of it in the forks ($1.29 against $1.86; fork cache writes 5.7k against 13.8k per fork).
- Quality held: graded 1/5 and 1/4 against 1/5 and 2/4, with about as many lens findings.
- The after arm also changed the fork boilerplate's read line, so forks review the files they inherited instead of re-reading them.
- Two runs per arm: a direction, not a measured effect size.

## Bench to discuss

The question (operator, 2026-10-13): with the same model as today's twill, does spawning after writing give (1) as good a result and (2) a lower cost in money and tokens?

- An A/B on Discussion-Write: spawn before writing against spawn after writing, the same reference records and lenses, more than one run per arm; the `twill-plan` bench is the precedent for the setup.
- Compare per arm: fork cache writes and reads, total cost, writer time, and the findings against the reference's Discussion-Review findings.
- The operator's choice for the test arm: spawn the forks after the files are written (2026-10-13).
- An idea that needs nested forks (operator, 2026-10-13): the writer forks once before writing; while the writer writes, that one fork waits, reads the written files, then forks the N lens forks, which inherit the files from it, so the files are read once instead of N times and the writer's drafting stays out of the lens forks' context.
  Claude Code refuses it: a fork holds the Agent tool, but a `subagent_type: "fork"` call from inside a fork fails with "Fork is not available inside a forked worker. Complete your task directly using your tools." (tested 2026-10-13).
- A third arm to consider: forks spawned early that read the code their lens needs while they wait, and read the written files only after the marker.
