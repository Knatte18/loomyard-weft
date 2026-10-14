# Card 1 — Record the weft push refusal's cause on PushWithRetry

**Edit:**
- `internal/gitrepo#PushWithRetry`

**Intent:** D4.
Add to the doc comment of `PushWithRetry` in `internal/gitrepo/pushretry.go` the finding: a retried `! [remote rejected] main -> main (failed)` refusal on the weft is the remote refusing a ref update that raced another update of the weft's `main` (board writes and other runs' record pushes), not a stale local ref, and the retry is the intended path for it.
State that the Warn line each retry logs is that race's signal and is kept on purpose.
One sentence per line, per `PATTERN-comment-line-breaks`.
No code, behavior or log level changes, so the existing `pushretry_test.go` tests stay as they are.

**ImpactSummary:** Comment-only change on one exported function; no callers, tests or behavior are affected.

**Commit:** 1: gitrepo: name the weft ref-update race behind a retried push refusal
