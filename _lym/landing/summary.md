# A start stopped by an unreachable origin waits on the dobby's catch-up

A task's first pass runs its own catch-up from origin before it claims the task.
When origin does not answer, that pass exits with `PassExitOriginWait`.
The dobby read that exit as a refused start.
It posted "dobby could not start the task" with the way forward `lym batten run <slug>` at once,
and backed the start off from one probe interval, doubling up to an hour.
A later pass with the same exit already counted as no failure and owed the catch-up,
so only a task's start stopped on a network blip and waited for someone to run it again by hand.

## What changes

- `firstPassExited` (`internal/dobbyengine/starts.go`) reads a first pass that exits `PassExitOriginWait` as owing the dobby's catch-up from origin.
  It records no start refusal, posts no notice and sets no backoff.
  The catch-up retries on its own delay, from a second doubling to the idle ceiling,
  and the task starts on the tick after the catch-up succeeds.
- `startReady` starts no task while the catch-up is owed.
  The tick already held its scheduling while a catch-up was owed at its start.
  This guard also covers a first-pass exit read earlier in the same tick,
  so the task is not started again before the catch-up has run.
- A catch-up waiting on origin posts no notice of its own, so before this change an origin that stayed down never reached anyone.
  `CatchUpWait` now keeps `Since`, when failures of its reason began.
  The prime's push summary lists a catch-up waiting on origin among the network waits as
  "the dobby's catch-up from origin for <n>m".
  It posts once the wait passes the summary's threshold, then posts the all-clear once the catch-up succeeds, as it does for an owed push.
  A record written before `Since` existed reads its `At` as the start of the wait.
- The package doc of `internal/dobbyengine` states both rules.

## Design decisions

- The exit code also covers a catch-up whose registration was refused or that failed for another reason.
  A first pass with any of those owes the catch-up as well.
  The dobby's catch-up then classifies the failure itself and posts its own notice for a refusal or another failure, once per episode.
  So no case loses its report, and the start is not refused twice over.
- `lym batten run` run by hand still refuses on an unreachable origin and names the re-run.
  Only the dobby's handling of that exit changes.
- The catch-up wait reaches the push summary only on the prime's dobby, the one the summary is wired on.
  A beam's dobby waits on the same origin, so the prime's dobby, waiting for the same reason, reports the episode.

## Tests

- `TestStartReady_RefusalsAndBackoff`, new subtest: a first pass exits `PassExitOriginWait` while origin stays down.
  The test checks that no notice and no start record appear, that the catch-up wait reads `origin`,
  and that the task's second start comes on the tick after the catch-up succeeds.
  Before the fix, the first notice was posted at once.
- `TestPushSummary`, new subtest: a catch-up failing on origin produces no summary before the threshold.
  At the threshold, one summary names the catch-up with its minutes counted from the first failure.
  The all-clear follows once the catch-up succeeds.

## Merge danger

**Door:** two-way
**Blast radius:** dobby
