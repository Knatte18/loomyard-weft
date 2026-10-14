# Card 2 — push-retry-finding

**Edit:**
- `internal/gitrepo#PushWithRetry`
- `internal/gitrepo#TestPushWithRetry_RetriesOnlyRetryableRefusalsUpToTheBound`

**Intent:** D4: record the weft push finding where the next reader of a retry log line looks.
No behavior and no log level changes; the Warn stays.

Add to `PushWithRetry`'s doc comment in `internal/gitrepo/pushretry.go`, after its existing sentences, that a `! [remote rejected] main -> main (failed)` refusal on the weft's `main` is the remote refusing a ref update that raced another update of the same ref (board writes and other runs' record pushes), not a stale local ref, and that the retry this function runs inside the push call is the intended path for it, so its retry log line is the race surfacing, not a fault.
One sentence per line, per `PATTERN-comment-line-breaks`.

The decision record's criterion names the doc comment itself, so a subtest pins it: `TestPushWithRetry_RetriesOnlyRetryableRefusalsUpToTheBound` gains the subtest "the doc names the weft main race as the retried refusal's cause", which parses `pushretry.go` with `go/parser` (`parser.ParseComments`), takes `PushWithRetry`'s `Doc.Text()`, and asserts it contains `` ! [remote rejected] main -> main (failed) `` and the word `race`.
It fails if the finding is dropped from the comment or names another cause; `internal/gitrepo`'s top-level test count is unchanged.

**ImpactSummary:** one doc comment in gitrepo and a subtest pinning it; no code or caller changes.

**Commit:** 2: Name the weft main ref-update race in PushWithRetry's doc
