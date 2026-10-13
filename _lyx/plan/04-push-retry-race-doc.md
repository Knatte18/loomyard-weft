# Card 4 — Record the weft push refusal's cause in the push retry's doc comment

**Edit:**
- `internal/gitrepo#PushWithRetry`

**Intent:**
D4: the doc comment of `PushWithRetry` in `internal/gitrepo/pushretry.go` gains the finding, so the next reader of a retry log line learns its cause there.
The comment quotes `! [remote rejected] main -> main (failed)` and says that on the weft it is a ref-update race: the remote refusing a ref update that raced another update of the weft's `main` (board writes and other runs' record pushes), not a stale local ref.
The retry is the intended path for it, and the Warn each retry logs is kept as the signal that surfaced the race.
No code, behavior or log level changes, so no test is added or changed; the card's `Verify:` checks the comment names the refusal line and the race, and the package's existing `TestPushWithRetry_RetriesOnlyRetryableRefusalsUpToTheBound` keeps covering the retry itself.

**ImpactSummary:** a doc comment on one exported function; no caller, test or behavior changes.

**Commit:** 4: Name the weft push refusal's race in PushWithRetry's doc

**Verify:** grep -qF '! [remote rejected] main -> main (failed)' internal/gitrepo/pushretry.go && grep -qF 'ref-update race' internal/gitrepo/pushretry.go
