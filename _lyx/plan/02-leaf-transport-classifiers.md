# Card 2 — git and GitHub transport classifiers

**Create:**
- `plan:internal/gitexec#IsTransportFailure` -> `func IsTransportFailure(err error) bool`
- `plan:internal/githubclient#IsTransient` -> `func IsTransient(err error) bool`
- `internal/gitexec/transport.go`
- `internal/gitexec/transport_test.go`
- `internal/githubclient/transient.go`
- `internal/githubclient/transient_test.go`

**Uses:**
- `internal/gitexec#GitError`

**Intent:**
Each leaf package exposes its own classification of an infrastructure failure (decision record D2), so the translation into `internal/shedengine`'s mark (card 4) never text-matches outside the package that owns the failing call, and neither leaf gains a `shedengine` import.

`IsTransportFailure` (`internal/gitexec/transport.go`) reports whether `err`'s chain holds a `*GitError` for a remote operation that failed to reach the remote.
Both conditions must hold:
- The first non-flag argument in `Args` is `push`, `fetch`, `ls-remote`, `clone` or `pull`.
- The lower-cased `Stderr` contains one of a closed set of transport phrases, declared once as an unexported slice: `could not resolve host`, `connection refused`, `connection reset`, `connection timed out`, `operation timed out`, `network is unreachable`, `failed to connect to`.
Anything else is not a transport failure: a rejected or non-fast-forward push, an authentication failure, a missing ref, a local command, and an error with no `*GitError` in its chain.
The doc comment states that an unrecognised message format falls back to "not transient", today's behaviour.

`IsTransient` (`internal/githubclient/transient.go`) reports whether `err`'s chain holds a GitHub API failure that a retry a moment later is likely to clear.
It is true for a `*github.ErrorResponse` whose response status is 5xx, and for a transport-level `*url.Error` that is not a context cancellation or deadline.
It is false for every 4xx response, and explicitly false for `*github.RateLimitError` and `*github.AbuseRateLimitError`, since an immediate retry hits the same limit.
The import stays inside `internal/githubclient`'s leaf allowlist (stdlib and go-github only).

Tests are untagged and spawn nothing: they build `*GitError`, `*github.ErrorResponse` and `*url.Error` values directly.
- `IsTransportFailure`: a `push` and a `fetch` with each transport phrase are true, also when wrapped with `%w`.
  A `push` rejected as non-fast-forward, a `push` failing authentication, a `fetch` of a missing ref, a `status` whose stderr happens to contain a transport phrase, a plain `errors.New` and nil are false.
- `IsTransient`: a 502 and a 503 `ErrorResponse` and a dial-failure `*url.Error` are true, also when wrapped.
  A 404, a 422, a 403, a `RateLimitError`, an `AbuseRateLimitError`, a `*url.Error` wrapping `context.Canceled`, a plain error and nil are false.

**Commit:** 2: gitexec, githubclient: classify transport failures
