# Retry the archive-tag push on git's short tag name, with longer backoff

Fixes #804.

A pair teardown pushes the archive tag `archive/<slug>/<sha>` to the weft origin, and a transient GitHub `(failed)` on that push halted the run until someone re-ran it.
The push already went through `gitrepo.PushWithRetry`, but the retry never ran for it.

## Cause

`retryablePushRefusal` retries a `! [remote rejected] <src> -> <dst> (failed)` line only when the line names the pushed ref.
`namesRef` matched the line when it held the ref as given or in a longer `refs/heads/` or `refs/tags/` form.
`archiveWeftTip` passes the full `refs/tags/archive/<slug>/<sha>`, while git prints a pushed ref in its short form, `archive/<slug>/<sha>`.
The line never matched, so the first refusal returned as a non-retryable error.
The rejection class seen was the one `PushWithRetry` retries; only the ref matching missed it.

## Change

- `namesRef` also matches the short form of a ref given with a `refs/heads/` or `refs/tags/` prefix.
  A full tag ref still never matches a full branch line of the same name.
- `PushWithRetry` takes the backoff delays from its caller and attempts one push more than it is given.
  The plain push primitives pass `PushBackoffDelays()`, so they keep four attempts at 1 s, 2 s and 4 s.
- The archive-tag push uses its own `archiveTagPushDelays`: 2 s doubling to 32 s, six attempts, about a minute before jitter.
- The `gitrepo` and `fabricengine` package docs state the full-ref matching and the archive schedule.

## Design decisions

- Only the archive push gets the longer schedule.
  The plain primitives retry inside locks that board writes and other runs' record pushes wait on, so a minute-long backoff there would stall them.
  A teardown archives once and holds no such lock.
- The classifier fix lives in `gitrepo`, not in the archive caller passing a short name, so any caller that passes a full ref is retried.

## Tests

- `TestRetryablePushRefusal_ClassifiesByTheReasonOnThePushedRefsLine` gains rows for a short-form line against a full tag ref and a full branch ref, a full-form line against a full tag ref, and a full tag ref against a same-named branch line.
- `TestPushWithRetry_RetriesOnlyRetryableRefusalsUpToTheBound` gains a row that pins the caller's delays and attempt count.
- `TestPushWithRetry_RetriesARealRefusalOfATagPushedByItsFullRef` (integration) pushes a tag by its full ref to a bare remote whose `proc-receive` hook refuses `refs/tags`, and checks that the retry runs until its attempts run out.
  The integration file's remote setup moved into `newRefusingRemoteClone` and `hookRuns`, which the existing plain-push test now uses too.
