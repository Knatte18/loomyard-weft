# Trial the cuts that make a gate run link fewer new test binaries

`gate-fewer-binaries-trial` · Task · research, gate, testkit · done

## Seen

`docs/research/gate-fewer-launches-2026-10-31/report.md` counted the test binaries a gate run links and launches.
Go links one test binary per package per tier, since the tiers select their test files by build tag: 532 on a cold tree, 143 new to the cache on a warm whole-suite run.
On OSL-1046 the endpoint scanner holds each new binary about 10 s, so a change of mean fan-in, about 100 new binaries across three tiers, costs about 1000 s there.
Five packages are imported by every test binary through the shared `TestMain` path (`internal/testkit/tmuxkit`, `internal/proc`, `internal/lymtemp`, `internal/lock`, `internal/agentname`), so a change to one of them rebuilds every binary.

## Idea

Try the report's cuts in practice before any of them is made permanent (operator, 2026-10-31), as a bench in a worktree of its own, measured on OSL-1046 as well as Neve:
- One test binary per package for all tiers, the tier chosen at run time instead of by build tag: the largest saving the report rates medium risk.
- Cut the five universal imports out of the common `TestMain` path: low risk, helps only when one of them changes.
- Merge small leaf test packages into shared binaries within a tier.
- One binary for the whole module per tier is rated high risk, since packages then share one process and Go's per-package test cache is lost; bench it only if the others fall short.

Each trial reports the new binaries and scanner hold it saves and whether the test results stay the same.

## Bench (operator, 2026-11-01)

- Measured on Neve: the count of new test binaries a gate run links is the same on both machines, so OSL-1046's hold is that count times about 10 s; no agent runs on OSL-1046, and one check run there by the operator at the end confirms it.
- Trials, in this order, each in this task's own worktree, none of them landed:
  1. the five universal imports cut out of the common `TestMain` path;
  2. one test binary per package for all tiers, tier chosen at run time, tried on a sample of the ten packages with the most tiers, the saving scaled to the whole repository from that sample;
  3. merging small leaf test packages, only if 1 and 2 fall short.
  One binary for the whole module is not trialled.
- Cap: per trial, three gate runs on a warm tree before the change and three after; every run counts the new binaries linked and checks the test results are unchanged.
- The task lands a report under `docs/research/` with its scripts and tables, never the trial changes themselves.
