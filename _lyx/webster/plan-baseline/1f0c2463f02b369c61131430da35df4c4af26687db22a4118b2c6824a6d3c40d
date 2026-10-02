# Card 4 — The parent-review request store

**Create:**
- `internal/parentreview/doc.go`
- `internal/parentreview/store.go`
- `internal/parentreview/store_test.go`
- `internal/parentreview/seam_enforcement_test.go`
- `internal/parentreview#Store`
- `internal/parentreview#Round`
- `internal/parentreview#Store.Latest`
- `internal/parentreview#Store.BeginRound`
- `internal/parentreview#Store.RecordDelivered`
- `internal/parentreview#Store.AddNotify`
- `internal/parentreview#Store.RecordVerdict`
- `internal/parentreview#Store.WaitNote`

**Prosa:**
- `CONSTRAINTS.md`

**Uses:**
- `internal/state#UpdateJSON`
- `internal/state#ReadJSON`

**Intent:**
Create `internal/parentreview`, the owner of decision D3's on-disk review exchange, as a told-geometry package: every path arrives on `Store`, and it imports the standard library, `internal/state`, `internal/logger` and (from card 5) `internal/shuttleengine` only.

`Store` carries `Root` (the told `_lyx/reviews/parent-review` directory), `LockDir` (its told `.lyx` mirror) and `Now func() time.Time` (nil means `time.Now`).
Each round is `round-<N>/` under `Root`, holding `request.json` (slug, round, reviewer full name, opened-at, the two discussion paths, and a state of `open`, `expired` or `superseded`), `brief.md` (the rendered reviewer brief), `delivery.json` (delivered-at or the last failure reason, gate prompts carried, last-prompt time, waiting notifies, and whether the cap Warn was logged), `verdict.json` (approve or reject, recorded-at, consumed) and `review.md` (the copied review file).
Every write goes through `internal/state` under a lock file in `LockDir`, so the driver and a verb in another process never interleave a read-modify-write.

`Round` is the decoded view of one round directory: its number, directory, request (nil before opening), delivery and verdict.
`Latest` returns the highest-numbered round; the verbs, the closure and status act on it alone.
`BeginRound` marks the latest round `superseded` when it has a request but no verdict and is still open, then creates the next round directory empty.
The store also exports what the gate closure needs: open the request (writing `request.json` and `brief.md`), record a carried prompt (bumping the count and the last-prompt time, or taking one waiting notify), mark the request expired, mark the verdict consumed, and mark the cap Warn logged.

The verb-facing methods refuse with exported sentinels the CLI maps to refusal messages:
`RecordDelivered` refuses no open request (`ErrNoOpenRequest`: no round, no request, or a superseded latest round) and an expired one (`ErrExpired`), and stamps delivered-at, or records the failure reason and logs a Warn when `failedReason` is non-empty, leaving delivered-at empty.
`AddNotify` refuses no open request, a recorded verdict (`ErrVerdictRecorded`) and an expired request, and adds one waiting notify.
`RecordVerdict` refuses the same three, refuses a reject with a missing or empty review file (`ErrEmptyReviewFile`), copies a given review file to `review.md`, and writes the verdict unconsumed.
`WaitNote` returns a one-line note for the latest round when it has an open request without a verdict — the reviewer's full name, the open time, and `notice delivered at <t>` or `notice not delivered` with the last failure reason — and the empty string otherwise.

`store_test.go` covers each method's state transitions and every sentinel over a temp directory with a fixed clock, including superseding on `BeginRound`, round numbering after a completed round, and `WaitNote` for open, delivered, verdicted, expired and superseded rounds.
`seam_enforcement_test.go` is the import allowlist, modelled on `internal/loomshed`'s, failing on any `internal/lyxcwd` import.
`CONSTRAINTS.md`'s Told-Geometry Invariant bound-packages list gains `parentreview`.
`doc.go` states the package's role and the round layout.
