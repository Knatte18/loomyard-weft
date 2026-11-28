# A start stopped by an unreachable origin waits on the dobby's catch-up

A task's first pass runs its own catch-up from origin before it claims the task.
When origin did not answer, the dobby read the pass's exit as a refused start:
it posted "dobby could not start the task" at once and backed the start off, doubling up to an hour.
A network blip is now retried quietly, and a long outage is reported by the push summary, as an owed push is.

## What changes

- A first pass that exits `PassExitOriginWait` now makes the dobby owe its own catch-up from origin.
  It records no start refusal, posts no notice and sets no backoff.
  The catch-up retries on its own delay, from a second doubling to the idle ceiling,
  and the task is started again once it succeeds.
- Such an exit is still a refused start when the catch-up cannot answer it:
  the dobby has no catch-up, or one already succeeded since an earlier such exit of the same board entry.
  Any other exit of that task's first pass in between makes the dobby forget the earlier one.
- `lym batten run` now exits `PassExitCatchUpRefused` (81) when its catch-up fails for any reason but an unreachable origin,
  a refused registration included.
  Before, these shared `PassExitOriginWait` (80).
  The dobby treats exit 81 as before: a first pass is a refused start, and a later pass owes the catch-up and counts as no failure.
- The start step now checks for an owed catch-up itself,
  so a first-pass exit that makes it owed also holds the starts later in the same tick.
- The prime's push summary now counts a catch-up waiting on origin as a network wait:
  its own dobby's, and that of each live beam dobby.
  Such a catch-up posts no notice of its own.
- The summary's heading "runs waiting on the network" becomes "waiting on the network",
  and the all-clear now reads "every branch is pushed to origin and nothing waits on the network".
- The dobby's state record keeps `since` on a catch-up wait, when failures of its current reason began.
  A record without it counts from `at`.

## Design decisions

- Only an unreachable origin is treated as transient.
- The pass's catch-up covers its own task, while the dobby's covers the pairs that stand,
  so a failure particular to the new task need not show in the dobby's.
  That is why a second origin-wait exit after a successful catch-up is reported: it bounds the retry.
  It also means that an origin that drops again during the retried first pass is reported at once,
  without waiting for the summary's threshold.
- The memory of an earlier origin-wait exit lives in the dobby process only.
  A restarted or re-exec'd dobby allows one more quiet retry.
- A beam dobby that is not live is left out of the summary, since nothing would retry or clear its record.
- A re-exec now owes the catch-up while the state record holds a wait, so a recorded wait is retried.
  A fresh start drops the wait a stopped dobby left, so a new wait counts from its own first failure.
- `lym batten run` run by hand still refuses on an unreachable origin and names the re-run.

## To check by hand

- Check that the prime's push summary covers every live beam dobby's catch-up waiting on origin:
  `hubBeamCatchUpWaits` in `internal/beamcli/dobbydeps.go` and `summarisePushes` in `internal/dobbyengine/pushsummary.go`.
- Check that a first pass whose catch-up fails for any reason but an unreachable origin is still a refused start:
  `passExitCode` in `internal/battencli/drive.go` and `firstPassExited` in `internal/dobbyengine/starts.go`.

## Merge danger

**Door:** two-way
**Blast radius:** dobby
