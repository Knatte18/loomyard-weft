# Decision record — retry-archive-tag-push

## Goal

A transient remote refusal of the archive-tag push in a pair teardown is retried with a longer backoff before the teardown fails, so a GitHub `(failed)` on the tag push no longer halts the run for an orch turn.

## Scope

- `internal/gitrepo/pushretry.go`: the retryable-refusal classifier matches a `! [remote rejected]` line that names the pushed ref in its short form when the caller passed the full `refs/heads/` or `refs/tags/` ref.
- `internal/gitrepo/pushretry.go`: `PushWithRetry` takes the caller's backoff delays, so one caller can retry longer than the plain push primitives.
- `internal/fabricengine/archive.go`: the archive-tag push retries on its own longer schedule.
- The `gitrepo` and `fabricengine` package docs state the archive-tag push's schedule and the ref matching.

Out of scope: the plain push primitives' schedule, the classified pushes' backoff, and any reason other than `(failed)` and `cannot lock ref`.

## Decisions

### The classifier matches the short name git prints for a full ref

Decision: when the caller's ref carries a `refs/heads/` or `refs/tags/` prefix, a refusal line naming the ref without that prefix belongs to it.
Rationale: git prints pushed refs in their short form, so the archive push, which passes `refs/tags/archive/<slug>/<sha>`, never matched its own `archive/<slug>/<sha> -> archive/<slug>/<sha> (failed)` line and ran exactly once.
That is the rejection class actually seen, the triage's premise that the retry already ran is wrong, and this is its fix.
Bound: a full ref still matches only lines in its own namespace or its short form; a full tag ref never matches a full branch line.

### PushWithRetry takes its backoff delays from the caller

Decision: `PushWithRetry` takes a delay slice, attempts one push more than the slice holds, and sleeps each delay in order; the plain push primitives pass `PushBackoffDelays()` and keep their schedule.
Rationale: the plain primitives retry inside the caller's lock, where a long backoff stalls other writers, while the archive push runs once per teardown and can wait out a longer GitHub hiccup.

### The archive-tag push retries six times over about a minute

Decision: the archive push sleeps 2 s, 4 s, 8 s, 16 s and 32 s between six attempts, each plus the jitter `PushWithRetry` adds.
Rationale: the operator asked for a longer backoff; two failed runs before a third succeeded points at a failure window of seconds to a minute, and a teardown that waits about a minute is still cheaper than an orch turn.

## Constraints

- `PATTERN-gitexec-checked-call`: the tag push stays a `gitexec.Run` call.
- `PATTERN-fabric-git`: the archive push stays in `internal/fabricengine`.
- `PATTERN-test-speed`: tests inject the sleep and never wait out a real backoff.
- `PATTERN-comment-line-breaks`: one sentence per comment line.

## Auto-mode assumptions

- The GitHub refusal seen was `! [remote rejected] archive/<slug>/<sha> -> archive/<slug>/<sha> (failed)`, as the issue's bare `(failed)` reads; no stderr capture of the failed runs exists.
- A one-minute backoff is the longer backoff the operator asked for.

## Open risks

- If GitHub's refusal lasts longer than a minute, the row still fails and the re-run workaround stays the path.

## Acceptance criteria

- A `(failed)` refusal line in short form is retryable for a caller that passed the full `refs/tags/` or `refs/heads/` ref.
- A full tag ref does not match a refusal line for the same name under `refs/heads/`.
- `PushWithRetry` sleeps exactly the delays the caller passes and attempts one push more than their count when every attempt is refused.
- Against a real remote that refuses a pushed tag, `PushWithRetry` with the full tag ref retries until its attempts run out.
- The plain push primitives keep their four attempts at 1 s, 2 s and 4 s.
