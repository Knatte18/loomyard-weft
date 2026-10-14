# Card 1 — push-retry-race-doc

**Edit:**
- `internal/gitrepo#PushWithRetry`

**Intent:** Record D4's finding in the doc comment of `PushWithRetry` in `internal/gitrepo/pushretry.go`, and change no code.
Add sentences saying that a retried `! [remote rejected] main -> main (failed)` refusal of the weft's `main` is the remote refusing a ref update that raced another update of the same `main` (a board write or another run's record push), not a stale local ref.
Say that retrying inside the push call is the intended path for that refusal, and that the `Warn` the loop logs per retry is the signal that surfaced the race and stays at that level.
The retry bound, backoff and log level do not change.

**ImpactSummary:** Comment-only change to one exported function's doc; no caller, test or behavior is affected.

**Commit:** 1: Name the weft main ref-update race in PushWithRetry's doc
