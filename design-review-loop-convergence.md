# Review loop: lens fan for Discussion and Plan, judge-narrowed rounds, a lighter convergence rule

`review-loop-convergence` · Note · enhancement, burler, loom

## What happened

On 2026-10-08 the Discussion review segments of `plan-glyph-precision`, `shuttle-state` and `reed-view` all ran their five rounds and escalated to the parent with cause `budget`.
In each, the round-5 reviewer's verdict was APPROVED with no BLOCKING finding, every earlier key was resolved, and each round raised fresh keys on edge cases rather than repeating old ones.
The parent then recorded the remaining design calls as decisions and accepted, which another round would not have changed.

Two causes:

- The judge's decision rule (`decisionRuleMarker` in `internal/shedadapters/bouncerprompt.go`) says `CONTINUE` while the latest round carries any `design` finding at MEDIUM or worse, and the judge stencil counts no fix as convergence, because a fix is new text no fresh reviewer has seen.
  Every round's fixes so yield a fresh full review and fresh MEDIUM keys until the budget runs out; the reviewer's own verdict carries no weight.
- Discussion and Plan review run solo: one `sonnet[high]` reviewer per round, no lenses.
  Burler's fork-based cluster review (`df1c6ba33`: the lens and fan library in `burler.yaml`, the judge's `exclude_lenses` channel) has never been wired into `loom-recipe.yaml`; no review row sets `cluster-fan`.
  The cluster design is what lets a review run on a mid-strength model; a solo reviewer covers one slice of the subject per round, so fresh edge cases surface round after round.

## Proposal

- Turn on a cluster fan for the Discussion and Plan review rows (`cluster-fan` on the BurlerRound rows, `cluster_excludes: true` on their Bouncer rows, which `internal/loomrecipe/clusterexcludes_test.go` pins to agree).
  Webster review stays solo: its forks may run no git and cannot reach a diff-only subject (the recipe comment on that row).
- Round 1 runs the broad fan; the judge narrows it each bounce through `exclude_lenses`, dropping lenses whose area is settled, so later rounds cost less.
- A lighter convergence rule: `CONVERGED` when the latest round carries no BLOCKING finding and no MEDIUM `design` finding on a key that was already open in an earlier round.
  New MEDIUM keys in the closing round are not dropped: they go to the next producer through the decision record, as the parent did by hand on 2026-10-08.
  BLOCKING always continues.
- Pick the fan's review model against the measured cost: the cluster design assumes `sonnet[medium]` lenses; a solo reviewer on Discussion and Plan would need `opus`.

## Blocker: gate quiescence with forks

A hub quick fix adding the fan stopped at `TestNoGatedRowAuthorizesForkSubagents` (`internal/loomrecipe/gatequiescence_test.go`), which pins a deliberately declined decision: a gated run must not spawn forks, because a gate fires at turn end and turn-idle alone would no longer mean the agent is finished.
Since the reviewer/fixer split (572ab84e8) the recipe's `gates` reach only the fixer, which never forks, but the reviewer, the half that forks under a fan, carries its own review-parse gate (`ReviewGateEntry`), so the question stands for any fan.
`shuttle-state` makes a turn end with outstanding background work the Stop payload reports a wait rather than a finished turn (its D8); the task re-opens the declined decision on that ground, rewrites or narrows the test, and corrects Webster-Burler's "Single reviewer, matching both shipped segments" comment.

## Cost and time

`docs/research/session-fork-spike.md` measured Claude Code fork subagents with a full prefix cache hit: about 17K fresh compute per extra reviewer after one shared explore phase, against about 209K for a cold reviewer.
So a fan's extra lenses are cheap next to the explore phase, which is paid once per round.
The forks run in parallel, so a round takes about the explore phase plus the slowest lens plus consolidation; the spike measured no wall-clock time.
The task measures both against today's solo rounds, per round and per segment, before the fan becomes the default.
