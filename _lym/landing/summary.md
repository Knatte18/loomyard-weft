# Retry transient git and GitHub failures before a run stops

A brief GitHub or network drop no longer halts a run and sends the parent a question.
Run-start, resume and leftover-teardown origin reconciles retry an unreachable origin, a failed push is pushed again, and the shed driver waits and re-steps a stop a network failure left.
Only a failure that outlasts those waits reaches the parent, as a notice naming the failing remote operation.

Closes #193
Closes #203

## What changes

- `internal/gitexec`: `IsTransportFailure` recognises more in-transit failures.
  These are HTTP/2 stream resets, recv and send failures, empty replies, early EOF, sideband disconnects, TLS termination and gateway 502, 503 and 504 answers.
  A stderr holding an HTTP 4xx status is a refusal and is not transient, even when a disconnect phrase follows it.
  408 and 429 stay retryable.
- `internal/fabricengine`: `RunStartReconcileOptions` retries an unreachable origin every 10 seconds for a minute.
  `pushClassified` pushes again over the push backoff when a push fails in transit.
- `internal/battencli` and `internal/beamcli`: a run's start, its resume, a gone pair's re-adoption and the leftover teardown use those options.
  `cancel` keeps a single judge-only attempt.
- `internal/sheddriver`: a stop left by a git transport or GitHub API failure is re-stepped after waits of 15 seconds, 30 seconds and one minute.
  A stop still failing after the third wait is handed back with a notice naming the row, the failing remote operation and that the run resumes once the remote answers.
  `Deps.Wait` is a new optional seam; a nil `Wait` waits on the real clock and removes no rule-table row.
  `pattern/PATTERN-shed-driver.md` records this.
- `internal/landingshed`: `Finalize` treats a failed fetch of the parent's upstream after the GitHub merge as a transient failure for the driver to re-step.
  The re-entry finds the pull request merged and lands through the already-landed path.

## Design decisions

- The reconcile retries run under the hub's remote-refs locks, so the bound is a minute, the shortest that still waits out a brief drop.
  The driver's waits, about one and three quarter minutes in all, run outside those locks and carry the longer outage.
- Transport classification stays phrase-based on git's stderr.
  The 4xx guard keeps permanent refusals, such as authentication failures, from being retried.

## To check by hand

- Finalize's fetch of the parent's upstream after the GitHub merge: confirm a transient failure re-steps and lands through the already-landed path (`internal/landingshed/finalize_pr.go`).
- The run-start origin reconcile: confirm it retries over `RunStartReconcileOptions` on a run's start, resume and re-adoption (`internal/battencli`).
- The transport phrases in `internal/gitexec/transport.go`: confirm none matches a permanent refusal.
- The default wait of `Deps.Wait` in `internal/sheddriver/drive.go`: confirm the real-clock path.
  The tests exercise the injected seam.

## Merge danger

**Door:** two-way
**Blast radius:** sheddriver
