# Card 4 — Record the weft push refusal finding in PushWithRetry's doc comment

**Edit:**
- `internal/gitrepo#PushWithRetry`

**Intent:**
Decision D4 changes no behavior and no log level: the retry's Warn line stays.
`PushWithRetry`'s doc comment gains the finding, so the next reader of a retry line learns its cause there:
`! [remote rejected] main -> main (failed)` on the weft is the remote refusing a ref update that raced another update of the weft's `main`, such as a board write or another run's record push, not a stale local ref.
That refusal is the one this function retries inside the push call, and a retry log line for it is the intended path.
The comment follows `PATTERN-comment-line-breaks`: one sentence per line.

**ImpactSummary:** Doc comment only; no code, test or log changes.

**Commit:** 4: gitrepo: name the weft ref-update race behind a retried push refusal
