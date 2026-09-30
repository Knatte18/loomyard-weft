# Card 3 — githubclient classifies its own transient failures

**Create:**
- `plan:internal/githubclient#IsTransient` -> `func IsTransient(err error) bool`
- `internal/githubclient/transient.go`
- `internal/githubclient/transient_test.go`

**Intent:**
`internal/githubclient` exposes its own classification of a GitHub API failure that a retry a moment later is likely to clear (decision record D2), so card 5's translation never inspects GitHub errors itself.

`IsTransient` (`internal/githubclient/transient.go`) reports whether `err`'s chain holds such a failure.
It is true for:
- a `*github.ErrorResponse` whose response status is 5xx;
- a transport-level `*url.Error` other than a cancellation;
- `context.DeadlineExceeded` anywhere in the chain, bare or inside a `*url.Error`.
  The landingshed producers bound each GitHub call with their own `context.WithTimeout`, and go-github returns `ctx.Err()` itself when that deadline fires, so this is the timeout D2 counts as a transport failure.
It is false for `context.Canceled`, every 4xx response, and explicitly for `*github.RateLimitError` and `*github.AbuseRateLimitError`, since an immediate retry hits the same limit.
A cancelled step never depends on the `Canceled` exclusion alone: every landingshed arm checks its own context and returns the cancellation before any GitHub error.
The import stays inside `internal/githubclient`'s leaf allowlist (stdlib and go-github only).

Tests (`transient_test.go`, untagged, spawning nothing, building values directly):
- True: a 502 and a 503 `ErrorResponse`, a dial-failure `*url.Error`, a bare `context.DeadlineExceeded` and a `*url.Error` wrapping it, each also when wrapped with `%w`.
- False: a 404, a 422, a 403, a `RateLimitError`, an `AbuseRateLimitError`, a bare `context.Canceled` and a `*url.Error` wrapping it, a plain error, and nil.

**Commit:** 3: githubclient: classify transient API failures
