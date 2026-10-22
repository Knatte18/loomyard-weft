# Retry the archive-tag push on git's short tag name, longer backoff

A pair teardown's push of `archive/<slug>/<sha>` to the weft remote now retries a transient `(failed)` refusal, over a longer backoff, before it fails the row.
A GitHub hiccup on that push no longer halts the run for an orch turn.

The push already went through `gitrepo.PushWithRetry`, but the retry never ran for it.
Git prints a refused ref in its short form, `! [remote rejected] archive/<slug>/<sha> -> archive/<slug>/<sha> (failed)`, while `archiveWeftTip` passes the full `refs/tags/archive/<slug>/<sha>`.
The classifier never matched the printed line to the pushed ref, so the push ran once.

Closes #804

## What changes

- `gitrepo` (`namesRef`): a ref passed in full `refs/heads/` or `refs/tags/` form also matches its short form.
  A full ref still matches only its own namespace or its short form, so a full tag ref never matches a `refs/heads/` line of the same name.
- `gitrepo` (`PushWithRetry`): it takes the backoff delays from the caller, sleeps each in order, and attempts one push more than the slice holds.
  The plain push primitives pass `PushBackoffDelays()` and keep four attempts at 1 s, 2 s and 4 s.
- `fabricengine` (`archiveTagPushDelays`): the archive-tag push retries on 2 s doubling to 32 s, six attempts in all, plus the jitter `PushWithRetry` adds.
- The `gitrepo` and `fabricengine` package docs state the ref matching and the archive push's schedule.

## Design decisions

- The archive push gets its own schedule because the plain primitives retry inside the caller's lock, where a long backoff stalls other writers.
  A teardown archives once and holds only the pair's own teardown lock, which only that pair's detached pushes wait on, so waiting out about a minute beats failing the row.
- The schedule is a judgment call.
  No stderr capture of the failed runs exists, so the refusal is assumed to be the bare `(failed)` the issue reads, and a minute is taken as the longer backoff the operator asked for.
- Only `(failed)` and `cannot lock ref` stay retryable; no other refusal reason is added.
- A short-form line `v1` cannot be told from a branch `v1` in git's output.
  The classifier keeps the match to the exact ref the caller passed, so this is inherent to the printed format and not a new ambiguity.

## Tests

- Unit rows cover the short-form match for full tag and branch refs, and the no-match of a full tag ref against a `refs/heads/` line.
- Unit rows pin `PushWithRetry` on caller delays and on an empty slice, and a test pins the archive schedule.
- An integration test pushes a tag by its full ref to a real bare remote that refuses every tag, and checks the retry runs until its attempts are exhausted.

## Not run

- No run against GitHub: if its refusal outlasts about a minute, the row still fails and re-running `lyx batten run <slug> --window` stays the path.
