# Support log — retry-archive-tag-push

## Interview

A darn run holds no interview: the decisions come from the board entry, the operator decision it records and a reading of `internal/gitrepo/pushretry.go` and `internal/fabricengine/archive.go`.

## Rejected alternatives

- Lengthening `pushBaseBackoff` or `pushMaxAttempts` for every push: the plain primitives retry inside the caller's lock, and a minute-long backoff there stalls board writes and other runs' record pushes.
- Passing the short tag name from `archiveWeftTip` instead of fixing the classifier: it fixes only this caller and leaves any other full-ref caller silently unretried.
- A new `PushWithRetryDelays` beside an unchanged `PushWithRetry`: two functions for one loop with only two callers; changing the signature keeps one entry point.
- Retrying the archive push on any `(...)` reason: a hook decline or a permission refusal is not transient and should fail at once.

## Review rounds

_No rounds yet._

## Question ledger

- Is the seen refusal the class `PushWithRetry` retries? Self-picked: yes in its reason, but the archive caller's full ref never matched git's short-form line, so the retry never ran.
- How long a backoff? Self-picked: six attempts, 2 s doubling to 32 s, about a minute before jitter.
- Should the plain primitives get the longer backoff too? Self-picked: no, they retry under locks other writers wait on.
