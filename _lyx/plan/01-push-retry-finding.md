# Card 1 — push-retry-finding

**Edit:**
- `internal/gitrepo#PushWithRetry`

**Intent:** Record the weft push finding (D4) in `PushWithRetry`'s doc comment, and change nothing else.
Add sentences, one per line, saying that a retried `! [remote rejected] main -> main (failed)` on the weft's `main` is the remote refusing a ref update that raced another update of that `main` (board writes and other runs' record pushes), not a stale local ref.
Say that this loop is the intended handling of that race, and that the retry's Warn line stays as the signal that surfaces it.
No code, no log level and no other comment changes; `internal/gitrepo/doc.go` describes behavior, which is unchanged, so it is not touched.
The acceptance criterion is a statement in a doc comment, so no test pins it: the card's review checks the comment, and the package's existing `TestPushWithRetry_RetriesOnlyRetryableRefusalsUpToTheBound` still covers the unchanged loop.

**ImpactSummary:** Doc comment only on one exported function; no code, caller or test changes.
