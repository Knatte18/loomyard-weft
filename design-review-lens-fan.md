# Review lens fans for Discussion and Plan, narrowed by the judge each round

`review-lens-fan` · Task · enhancement, burler, loom · running · Batchifier

Depends on: [`shuttle-state`](design-shuttle-state.md), [`review-loop-convergence`](design-review-loop-convergence.md)

## Why

Discussion and Plan review run solo: one reviewer per round, no lenses.
Burler's fork-based cluster review (`df1c6ba33`: the lens and fan library in `burler.yaml`, the judge's `exclude_lenses` channel) has never been wired into `loom-recipe.yaml`; no review row sets `cluster-fan`.
The cluster design is what lets a review cover a subject broadly in one round; a solo reviewer covers one slice per round, so fresh edge cases surface round after round.
Review loops are the largest share of a run's agent time (see task `review-loop-convergence`).

## Proposal

- Turn on a cluster fan for the Discussion and Plan review rows (`cluster-fan` on the BurlerRound rows, `cluster_excludes: true` on their Bouncer rows, which `internal/loomrecipe/clusterexcludes_test.go` pins to agree).
  Webster review stays solo: its forks may run no git and cannot reach a diff-only subject (the recipe comment on that row).
- Discussion and Plan get their own fans in `burler.yaml`, with lenses written for a design record and a plan rather than the code-oriented `standard` lenses (error-handling, test-gaps).
- Round 1 runs the broad fan; the judge narrows it each bounce through `exclude_lenses`, dropping lenses whose area is settled, so later rounds cost less.

## Blocker: gate quiescence with forks

A hub quick fix adding the fan stopped at `TestNoGatedRowAuthorizesForkSubagents` (`internal/loomrecipe/gatequiescence_test.go`), which pins a deliberately declined decision: a gated run must not spawn forks, because a gate fires at turn end and turn-idle alone would no longer mean the agent is finished.
Since the reviewer/fixer split (572ab84e8) the recipe's `gates` reach only the fixer, which never forks, but the reviewer, the half that forks under a fan, carries its own review-parse gate (`ReviewGateEntry`), so the question stands for any fan.
`shuttle-state` makes a turn end with outstanding background work the Stop payload reports a wait rather than a finished turn (its D8); this task re-opens the declined decision on that ground, rewrites or narrows the test, and corrects Webster-Burler's "Single reviewer, matching both shipped segments" comment.

## Cost and time

`docs/research/session-fork-spike.md` measured Claude Code fork subagents with a full prefix cache hit: about 17K fresh compute per extra reviewer after one shared explore phase, against about 209K for a cold reviewer.
So a fan's extra lenses are cheap next to the explore phase, which is paid once per round.
The forks run in parallel, so a round takes about the explore phase plus the slowest lens plus consolidation; the spike measured no wall-clock time.
The task measures tokens and wall time per round and per segment against solo rounds before the fan becomes the default.

## Models

Operator, 2026-10-08: a fanned review's forks run on sonnet, while a solo reviewer runs on opus; code review (Webster-Review) runs on sonnet either way.
The fan's fork model is its own setting, separate from the segment's solo `review` model.
