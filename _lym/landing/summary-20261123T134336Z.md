# Wait out a transient network failure before a run's parent hears of it

GitHub dropping connections for a minute stopped several runs at once.
Each stop reached the parent orch as a question, and a plain re-run then went through.
This change fixes the two defects behind that.

## Classify dropped connections as transport failures

`gitexec.IsTransportFailure` knew only connect-phase failures: unresolved host, connection refused, reset or timed out.
A connection GitHub's front end dropped after it was made printed lines such as
`HTTP/2 stream 1 was closed cleanly, but before getting all response header fields`, and the classifier read them as a refusal.
The failure then carried no `git-transport` mark.
The shed loop's re-step and the origin reconcile's ls-remote retry never applied to it.

`transportPhrases` now also names an HTTP/2 stream close or reset, a curl receive or send failure, an empty reply,
an early EOF or sideband disconnect, a GnuTLS handshake or cut, and a 502, 503 or 504 from the front end.
A 403 and a 404 stay refusals.

## Re-step a network failure after waits, then notify

`sheddriver`'s rule table sent every transient stop that outlived the loop's one immediate re-step to the tackler.
The tackler, finding nothing to repair, handed it to the parent as a question.

A stop marked `git-transport` or `github-api` now gets two new rows ahead of that one:

- below the re-step cap, the driver waits and re-steps.
  The waits are `networkRestepWaits`: 15 s, 30 s, then a minute.
- past the cap, the driver hands the run back with no tackler call.
  The parent's notice names the row, the waits and the failing remote operation as the stop's error quotes it,
  and says the run resumes once the remote answers.
  It carries no question.

The re-step count is the driver's existing per-stop-key settle-record count.
A restarted driver therefore keeps it, and a wake stamp restarts it as before.
A changed on-disk build still hands over first.
Other transient classes, `agent-start` and `host-resources`, still go to the tackler.

The wait runs through a new optional `Deps.Wait` seam.
Unwired, it waits on the real clock and ends early with the context, so neither the loom nor the batten driver wiring changes.
The wait comes before the pause hold and the status recheck,
so a pause or an orch goto made during the wait is still honoured before the re-step.

## Design decisions

- The retry sits in the driver, not inside `gitexec`.
  A gitexec-level backoff would make every test that points git at a refused origin sit out the backoff for real,
  and gitexec has no cross-package seam to shorten it.
  It would also re-run pushes whose outcome on the remote is unknown after a dropped connection.
  The driver's seam is injectable, and a re-step re-runs a whole step that is already safe to repeat.
- GitHub API failures share the rows: the loop already marks them with the same class of cause, a remote that did not answer.

## Not covered

An origin reconcile run with zero `ReconcileOptions` still makes one ls-remote attempt before it reports origin unreachable.
That covers `lym batten run`'s catch-up and the owner checks.
Those callers fall back to the local board or refuse with "origin is unreachable", outside any shed step, so no driver re-step reaches them.
Giving them a production retry bound would make the integration tests that point origin at a refused URL wait it out.
They need a seam first, as a follow-up.

## Tests

- `TestIsTransportFailure` gains a row for each new phrase, the issue's HTTP/2 line among them, across push, fetch and ls-remote,
  and rows pinning a 403 and a 404 as refusals.
- `TestDecideDrive` pins each wait in turn, the hand-back with the network notice once three re-steps were made,
  the hand-over on a changed build, and another transient class still going to the tackler.
- `TestGoDriver_Settle` scripts a network failure that outlasts every wait.
  It pins the three waits between four steps, three re-step records, one hand-back, and the exact notice with no question.

## Merge danger

**Door:** two-way
**Blast radius:** sheddriver
