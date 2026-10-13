# Twill: spawn lens forks before or after writing

`twill-fork-timing` · Note · audit, loom, stencil

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

## Plan bench, wave 1 (2026-10-13): stopped on a shuttle bug

The Plan-Write bench (spawn-after arm against `twill-plan` Round 2's spawn-before arm, `opus[medium]`) stopped after wave 1: in both runs shuttle ended the writer mid-turn after the last fork finished, so no lens review was written (task `shuttle-gate-early-done`).
Cost so far, without the writer's review phase: forks $1.16 and $0.95 per run against $1.64, fork cache writes 6.2k and 3.8k against 16–24k; total spend about $10.3 of a $25 cap.
The operator holds further benches until `shuttle-gate-early-done` lands (2026-10-13); then four fresh spawn-after runs.
`shuttle-gate-early-done` landed 2026-10-13 (`b1c731a2`).

## Decided (operator, 2026-10-13)

Spawning the lens forks after the files are written goes in; further benches refine it, they do not decide it.
The wave-1 pairs are torn down (2026-10-13); the evidence stays in the hub prime's `.scratch/twill-fork-timing-plan/`, and the next bench sets up fresh pairs.

## Arm to bench: a writer fork (operator, 2026-10-13)

With the forks spawned after writing, every lens fork inherits all the writer did to reach the files: drafts, write calls, failed `validate-plan` rounds.
The idea:

1. The master explores until it holds all it needs to think and write the files.
2. It forks one fork0, which thinks, writes the files (running `validate-plan` itself) and ends.
3. The master reads the written files: it sees them fresh, on top of everything it explored, blind to the reasoning behind them.
4. The master forks the lens forks as today; they inherit the exploration and the files, not the drafting.

Allowed: the master forks both times, so the fork-cannot-fork limit does not apply.
Points to measure or settle:

- The shared prefix likely stays warm on its own, since every fork0 turn reads it; the operator's idea of `lyx shuttle wait` to keep the master's cache warm may not be needed, and the bench should check.
- The lens forks' inherited context gets smaller, so they should cost less than in the after arm.
- Fork0 has ended when the findings come back, so the master fixes them without the reasoning: it fixes against the files, not the intent, which may help or may miss why something was written so.

Bench it as a third arm against the after arm: the same reference, two runs per arm, cost and findings per arm.

Measured from existing spawn-after transcripts (2026-10-13, scripts in the hub prime's `.scratch/drafting/`): the drafting the master would shed is 2–4 % of its context at the first fork spawn in Discussion and 5.5–12 % in Plan (12–26k tokens), about two thirds of it the write turns and half of that thinking.
At cache-read prices that is $0.003–0.016 per fork and $0.03–0.09 per run gross; the master's Read of the finished files and fork0's own reads take most of it back, so the net cost effect is about zero.
The case for the arm is quality, lens forks that judge the files blind to the reasoning, not cost.

## Plan bench, after against fork0 (2026-10-13)

Two runs per arm on the `run-time-by-role` reference, `opus[medium]`, $18 in all; report `docs/research/twill-fork-timing-plan-bench.md`, evidence in the hub prime's `.scratch/twill-fork-timing-plan/`.

- Cost per plan: after $4.60 mean, fork0 $4.40 mean; lens forks $0.94 per run in both, against $1.60 in the spawn-before runs, so spawn-after holds its saving.
- The master's cache stayed warm through fork0's run (about 5 minutes): `lyx shuttle wait` is not needed.
- Quality on the key: level or slightly worse for fork0; the fork0 master once followed a wrong finding and once dropped a detail, missing fork0's intent.
- fork0's blind lens forks were sharper: more BLOCKING and MEDIUM findings (6, 6 against 2, 4), one a real BLOCKING the after arm shipped.
- Caveats: one reference, the prototype route (`plan_producer: single` with a spliced directive), not the twill producer.

Bench verdict: no cost gain, no case to build fork0 yet; if benched again, fork0 writes a short rationale file the master reads before fixing and the lens forks do not inherit.

## Next arm to bench: fork0 stays alive and revises (operator, 2026-10-13)

The fork0 arm's weak point: fork0 has ended when the findings come back, so the master fixes without fork0's reasoning.
The operator's idea: fork0 never ends, and revises the plan itself, as the writer does today, while the lens forks stay blind to its reasoning.

1. The master explores and forks fork0.
2. fork0 writes the plan, runs `validate-plan` and writes the ready marker.
3. The master awaits the marker and forks the lens forks; they inherit the master's context and the written files, not fork0's reasoning.
4. Each lens fork writes its findings to its own file (`lens-<name>.md`) instead of returning them to the master, so the master never reads them.
5. fork0 runs a Go command that waits for every lens file, merges them mechanically into one findings file and returns; it re-runs it like `lyx shuttle await`, under the cache lifetime, so fork0's cache stays warm while it waits.
6. fork0 judges the merged findings with its own reasoning, rejects wrong ones with a reason, revises, and ends.

No dummy file is needed: the merged file exists only once the command has written it.
Lens forks are read-only today and return findings as their final message (`contracts/stencils/twill/twill-directive.md`), so the arm needs each fork's write permission limited to its own lens file, and the merge command (for instance a `lyx twill` verb taking the expected lens files).
Prototype the merge with a scratch script for the bench before building it.

Fallback, if keeping fork0 alive fails: fork0 writes a second file with its rationale, which the master reads only after it has forked the lens forks, then fixes as in the fork0 arm; the lens forks are told not to read it.

Measure against the after arm and the fork0 arm: cost per plan, fork0's await calls and cache writes, lens findings, rejected findings and the grade on the key.

## Plan bench, fork0 stays alive (2026-10-13)

Two runs (`twb-ftr-1`, `-2`), same reference and setup, $10.74; all six plans of both rounds re-graded by one fresh grader session; report section "Round 3" in `docs/research/twill-fork-timing-plan-bench.md`, evidence and the merge script in the hub prime's `.scratch/twill-fork-timing-plan/alive/`.

| Arm | Total per plan | Lens forks | Key repeated + partly, both runs |
|---|---|---|---|
| after | $4.60 | $0.94 | 8 |
| fork0 | $4.40 | $0.94 | 11 |
| alive | $5.37 | $1.33 | 9 |

- The mechanism works: one findings file per lens, a mechanical merge, one wait call of about 2 minutes, and fork0's cache stayed warm (about 300 tokens written on waking against 184–205k read).
- Fixing with fork0's own reasoning did not help: ftr-1 repeated both of the fork0 arm's misses (J1, J5); fork0 rejected 0 and 2 findings, both rejections right.
- The extra cost is fork0's revise phase ($0.56–0.78 per run) and lens forks that inherited more context.

Bench verdict: no quality gain for $0.75–1.00 more per plan; the after arm (writer fixes its own plan) stays the best choice, and neither fork0 variant is worth building on this evidence.

Ideas from the bench agent, untested: merge two lenses (for instance `test-economy` into `row-reachability`), have forks batch their reads into one or two calls, and cut the writer's `validate-plan` rounds (the writer is about $4 of $5.6 per plan).
