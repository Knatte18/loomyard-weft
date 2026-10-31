# Card 14 — shared-writer-wrapper

**Create:**
- `internal/loomshed#runAndCommit`

**Edit:**
- `internal/loomshed/doc.go`
- `internal/loomshed#discussionWrite.Call`
- `internal/loomshed#darnWrite.Call`

**Intent:** The writer wrappers of Discussion-Write and Darn-Write share one wrapper for running a session and committing its outputs (D8); the prior-work respawn stays darn's.

`runAndCommit` calls `session.Call(ctx)` once and returns its triple when the error is non-nil or the pointer's Path is empty; otherwise it refuses a nil `commit` with the named error `loomshed: <name>: no commit seam wired; the produced output would never be committed`, calls `commit`, maps a commit error to the returned error `loomshed: <name>: commit produced artifacts: <err>; way forward: the fault is transient, re-step the <name> row`, and returns `(outcome, pointer, nil)`, a gate-failed Stuck included, so a halted run leaves the work committed.
`discussionWrite.Call` is that call over its inner producer and commit seam; `darnWrite.Call` reads the latest outcome, builds the session with the prior-work note, and calls `runAndCommit` over its landing-directory commit, then keeps its own steps after it (the resume way forward on a gate-failed Stuck and card 13's guard). The messages each wrapper returns today are kept, so `TestDiscussionWrite_Call` and `TestDarnWrite_Call` pass unchanged and cover the shared path.

**ImpactSummary:** A refactor inside loomshed: two producers' Call bodies share one helper, with identical outcomes and messages.
