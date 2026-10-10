# Loop machinery small bugs: bounce budget, PR-Gate comment, sleep-aware idle, skill-load hold

`loop-small-bugs` · Task · bug, loom, burler, shed, batten, shuttle · running · PR-Gate

## Goal

Five small faults in the loop machinery: the bounce budget, the PR-Gate budget comment, idle notices after sleep, hold notices at skill load, and a shuttle state disagreement.

## 1. Bounce budget blocks an approved round (#503)

In `publish-recovery` (2026-10-10) Webster-Review's round 3 was APPROVED and its fixer finished with a green verify gate, but the run blocked with "bounce budget exhausted for Webster-Burler" before the Bouncer wrote `round-3-bouncer-verdict.md`; `lyx loom circling accept` refused because the run was blocked, not awaiting, and the workaround (goto Webster-Burler plus resume) spent another review round on approved work.
Expected: the budget is checked before a round starts, not after an approved round ends, so an approved round always reaches its Bouncer verdict and the run moves on.

## 2. PR-Gate budget comment (#526)

A comment in `contracts/recipes/loom-recipe.yaml` says PR-Gate's "episode Stuck count never resets between rounds, so the budget is five rejection rounds per task", but `episodeStuckCount` in `internal/shedengine/run.go` ends the episode on a Done by any row in the segment, and `PR-Rework` shares the segment with `PR-Gate`, so the count resets after every rework and `max_bounces: 5` never trips.
An agent trusted the comment and wrote a five-rejection bound into a design.
Expected: the comment states the engine's real behaviour, in every recipe that carries it; a per-task PR-Gate budget is out of scope.

## 3. Idle notices count machine sleep

After a hibernate (2026-10-08 22:17 to 2026-10-10 06:01) batten raised an idle notice for every run ("agents idle for 31h46m") although every agent had resumed and was working.
Expected: the idle clock measures awake time (a clock that stops during suspend, `CLOCK_MONOTONIC` on Linux, rather than wall time, or a gap matching a sleep window is dropped), so after a wake a run is idle only once its agents stay quiet for the threshold of awake time.

## 4. Hold notice at the skill-load turn end (#485)

The orch got a hold notice for `ly:bolt-and-store:discussion` whose last message was «ok scribe:prose, scribe:conversation», the reply to shuttle's own skill-load prompt; the start prompt was typed right after and the agent worked within seconds.
Expected: a turn end answering the skill-load prompt, or any turn shuttle injected before the start prompt, never counts as a task turn end for the hold check; a hold notice is raised only after the start prompt was sent.

## 5. Shuttle logs held and busy disagreeing

In `cli-surface` the Webster step trace logged `shuttle: session state disagrees (loop=held, state=busy)` before idle-done; no stop followed.
Expected: find whether held and busy can disagree for long; the log line names which side wins, and a lasting disagreement resolves to one state.
