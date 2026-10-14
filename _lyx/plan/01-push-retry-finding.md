# Card 1 — push-retry-finding

**Edit:**
- `internal/gitrepo#PushWithRetry`

**Intent:**
Record decision D4's finding in the doc comment of `PushWithRetry` in `internal/gitrepo/pushretry.go`, and change no code.
The doc comment gains sentences naming the cause of a retried `! [remote rejected] main -> main (failed)` refusal on a weft push.
The cause is the remote refusing a ref update that raced another update of the weft's `main`, such as a board write or another run's record push, and not a stale local ref.
The comment says this retry is the intended path for that refusal, and that the Warn each retry logs is kept on purpose as the signal that surfaces the race.
No behavior or log level changes, so no test changes.

**ImpactSummary:** One doc comment in `internal/gitrepo/pushretry.go`; no code, caller or test changes.
