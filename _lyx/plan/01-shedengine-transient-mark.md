# Card 1 — shedengine transient mark

**Create:**
- `plan:internal/shedengine#TransientClass` -> `type TransientClass string`
- `plan:internal/shedengine#TransientGitTransport` -> `const TransientGitTransport TransientClass = "git-transport"`
- `plan:internal/shedengine#TransientGitHubAPI` -> `const TransientGitHubAPI TransientClass = "github-api"`
- `plan:internal/shedengine#TransientAgentStart` -> `const TransientAgentStart TransientClass = "agent-start"`
- `plan:internal/shedengine#MarkTransient` -> `func MarkTransient(class TransientClass, err error) error`
- `plan:internal/shedengine#TransientOf` -> `func TransientOf(err error) TransientClass`
- `internal/shedengine/transient.go`
- `internal/shedengine/transient_test.go`

**Edit:**
- `internal/shedengine/shed.go`
- `internal/shedengine/status.go`
- `internal/shedengine/run.go`

**Intent:**
Declare the transient mark once, in `internal/shedengine` (decision record D1, D2, D12), stdlib-only per the Shed Producer-Seam Invariant.

`transient.go` holds `TransientClass`, its three constants (the closed class set, stated once here in the type's doc comment: `git-transport`, `github-api`, `agent-start`, with one line each on what qualifies and what never does), an unexported error type carrying the class and the wrapped error, `MarkTransient` and `TransientOf`.
`MarkTransient` returns `err` unchanged when the class is empty or `err` is nil, and otherwise wraps it so that `Error()` returns the wrapped error's text verbatim and `Unwrap` returns the wrapped error — the status file's `error` text and every `errors.Is` match stay exactly as before.
`TransientOf` finds the mark anywhere in the chain via `errors.As` and returns its class, or the empty class when there is none.

`Shed` (`shed.go`) gains a `Transient func(error) TransientClass` field: the caller-supplied classifier the producer boundary uses to translate a lower-level failure into the mark.
Nil means no error is ever classified, which keeps every existing caller and test unchanged.
Its doc comment says the classifier is told, like `CommitStatus`, so this package never imports the packages whose failures it classifies.

`Status` (`status.go`) gains `Transient string` with JSON tag `transient,omitempty`, so a status file written before the field existed still decodes, under the strict read gate included.

In `Shed.stepLocked`'s `callErr != nil` arm with a healthy context (`run.go`), run the classifier when it is set and a class comes back non-empty, re-wrap `callErr` with `MarkTransient`.
Then persist `failed` with that class, and return the marked error, so a caller reads the class with `TransientOf`.
`persist` writes `Transient` on every call: the class on that arm's write, and the empty string on every other write (the pause writes, the step-3b resume write, every verdict write, and the unrecognised-outcome `failed` write), so a later step never carries a stale class.
A gate verdict (`Stuck`, `Awaiting`, a pause) never runs the classifier.

Tests in `transient_test.go`:
- `MarkTransient` then `fmt.Errorf("...: %w")` round-trips through `TransientOf`, preserves the error text exactly, and keeps `errors.Is` against the inner sentinel.
- An empty class and a nil error both leave the error unmarked; `TransientOf(nil)` and an unmarked error give the empty class.
- A `Step` over a producer returning an error, with a classifier answering `TransientGitTransport`, persists `state: failed` with `transient: "git-transport"`, and the returned error's `TransientOf` is that class.
- The same step with a classifier answering empty persists an empty `transient`.
- A later step that resumes the failed run and gets a `Done` verdict clears `transient`.
- A `Stuck` verdict, an `Awaiting` verdict and a pause never call the classifier (a counting classifier stays at zero) and persist an empty `transient`.
- A nil `Transient` field leaves a producer error unmarked.

**ImpactSummary:** Adds one optional field each to `Shed` and `Status` and one classify call in the producer-error arm; every current caller leaves `Transient` nil, so behaviour is unchanged until card 7 sets the classifier in `shedbuild.NewShed`.

**Commit:** 1: shedengine: declare the transient mark and persist it with failed
