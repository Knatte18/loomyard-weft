---
round: 2
focus:
  - "contracts/stencils/beam/beam-template-describe.md: does the stencil only point at `loom-final-summary-spec.md` for the summary format, restating no title-line syntax, length cap or Merge danger section wording, as PATTERN-stencil-ownership requires?"
  - "internal/dobbyengine/loop.go `daemon.tick` and internal/dobbyengine/describe.go `describeStep`, `spawnDescribe`, `settleDescribe`: when the describe step fails, on a spawn error or an unreadable draft, does the board tidy and the beam's landing step still run, is the attempt cleared past its deadline, and is the orch told once?"
  - "internal/dobbyengine/describe.go `settleDescribe` and internal/beamcli/dobbydeps.go `beamDescribeDeps` Record: is a rewrite that commits but whose pull request update fails reported as recorded with a way forward for the pull request, and is a failed post of the failure line retried without calling Record again?"
  - "internal/dobbyengine/loop.go `exitWorktreeGone` and the kept-agent readers: is an in-flight describer agent stopped when the beam's worktree is gone, as the board-tidy agent is?"
  - "internal/landingshed/beamdescribe.go `ReplaceBeamDescription`: when writing the draft fails partway, does the recorded description stay intact?"
  - "_lym/landing/summary.md `## Merge danger`: does the door call and blast radius match what dobby now does on its own, namely committing the description, pushing the beam branch and editing the pull request body?"
  - "internal/dobbyengine tests: does a tick-level test set `Deps.BeamDescribe` so that removing `describeStep` from `tick`, or running it under an anchor-move, adopt or catch-up hold, fails a test?"
---

Round 1 ended with one BLOCKING finding and several MEDIUM design findings, all on the describe step.
Fixes made since are new text; the entries above point the next review at each site they touch.
