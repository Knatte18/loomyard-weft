# Card 8 — push-auth-class

**Create:**
- `plan:internal/gitrepo#ErrPushAuthRefused` -> `var ErrPushAuthRefused error`
- `plan:internal/gitrepo#authRefusalTriggers` -> `var authRefusalTriggers []string`
- `plan:internal/gitrepo#PushBackoffDelays` -> `func PushBackoffDelays() []time.Duration`

**Edit:**
- `internal/gitrepo#Repo.PushRebaseFree`
- `internal/gitrepo_test#TestPush`
- `internal/gitrepo#TestPushWithRetry_RetriesOnlyRetryableRefusalsUpToTheBound`

**Intent:** D10's auth or permission class needs a push error distinguishable from a rejection and a transport failure.
`Repo.PushRebaseFree` checks git's stderr for an auth or permission refusal (`authRefusalTriggers`: GitHub's `Permission to`, `denied to`, `returned error: 403`, `Authentication failed`, `could not read Username`) before its rejection check, and returns an error that wraps `ErrPushAuthRefused` with git's last error line, matched with `errors.Is`; the bounded `(failed)` retry never retries it.
The open risk the record names holds: a wording change leaves a 403 matching neither `authRefusalTriggers` nor `gitexec.IsTransportFailure`'s phrases nor a rejection line, so it reaches card 9 in no class it lists, as an unclassified push error.
`PushBackoffDelays` returns the doubling delays `PushWithRetry` sleeps between its attempts, derived from `pushBaseBackoff` and `pushMaxAttempts` and never from a second literal, so card 9's gone-path backoff in fabricengine sleeps the same delays.

`TestPush`, the existing step test of `PushRebaseFree`'s error classes (integration, package `gitrepo_test`), gains a step over a local bare whose `pre-receive` hook prints GitHub's 403 wording and exits non-zero, which asserts `errors.Is(err, ErrPushAuthRefused)`, not `ErrPushRejected` and not `ErrPushRetriesExhausted`, and that the hook ran once.
`TestPushWithRetry_RetriesOnlyRetryableRefusalsUpToTheBound`'s persistent-refusal row also asserts that its recorded sleeps equal `PushBackoffDelays()`.

**ImpactSummary:** `PushRebaseFree`'s callers all live in fabricengine; today they treat the new error as an unclassified push error, and card 9 gives it its class; `PushBackoffDelays` is new and has no caller until card 9.

**Commit:** 8: gitrepo: an auth refusal of a push is its own class
