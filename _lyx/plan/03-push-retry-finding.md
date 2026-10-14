# Card 3 — push-retry-finding

**Edit:**
- `internal/gitrepo#PushWithRetry`

**Intent:**
D4: record the weft push finding in `PushWithRetry`'s doc comment in `internal/gitrepo/pushretry.go`, with no behavior or log-level change.

The comment gains the cause of a retried `! [remote rejected] main -> main (failed)` refusal: the remote refused a ref update of the weft's `main` that raced another update of the same ref (board writes and other runs' record pushes), not a stale local ref.
It states that the in-call retry is the intended path for that refusal and that the Warn each retry logs stays, since that line is the signal that surfaced the race.
So the next reader of a retry line learns its cause at the function that logged it.

The change is a doc comment only, so it carries no test: no behavior changes for a test to pin, and the existing `gitrepo` tests keep covering the retry itself.

**ImpactSummary:** Doc comment only; `PushWithRetry`'s behavior, callers and log output are unchanged.
