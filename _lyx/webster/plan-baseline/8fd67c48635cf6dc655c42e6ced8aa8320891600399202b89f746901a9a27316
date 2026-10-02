# Card 4 — review-store-rounds

**Create:**
- `internal/parentreview#Store.RejectedRounds`
- `internal/parentreview#Store.PrepareRound`
- `internal/parentreview#Store.SupersedeCapReject`
- `internal/parentreview#ErrNotAtCap`

**Edit:**
- `internal/parentreview/store.go`
- `internal/parentreview/doc.go`
- `internal/parentreview/store_test.go`

**Intent:**
Give the round store what multi-round review needs, so the cap is decided by what is on disk and survives an attach, a driver restart or a resume.

- `Request` gains `Cap int` (`cap`, `omitempty`): the reject cap the gate held when it opened the round.
  `OpenSpec` gains `Cap`, and `OpenRequest` writes it.
- `Verdict` gains `Superseding bool` (`superseding`, `omitempty`): set only on an approve that replaced the cap's reject.
- `RejectedRounds` counts rounds whose verdict kind is reject, across every round under `Root`.
- `PrepareRound` is Discussion-Write's fresh-spawn preparation: it returns the latest round unchanged when its verdict is a reject or a superseding approve, and otherwise calls `BeginRound` exactly as today.
- `SupersedeCapReject` replaces the latest round's reject with `Verdict{Kind: VerdictApprove, Superseding: true, RecordedAt: now}` under that round's lock, keeping `review.md`.
  It refuses with `ErrNotAtCap` unless, read under the lock, the latest round's verdict is a reject, its request's `Cap` is positive, and `RejectedRounds` (counting that reject) is at least that `Cap`.
  It never rewrites an earlier round and never touches an open round's pending state.
  The run-status half of the bound (the run is `blocked`) is the CLI's check in card 10, since the store knows no run state.

Update `doc.go`'s file table for the new `cap` and `superseding` keys and add one sentence that rounds repeat after a reject until the cap.

Tests in `store_test.go`: `RejectedRounds` over a mix of approve, reject, expired and empty rounds; `PrepareRound` keeps a reject round and a superseding-approve round and begins a new round after a plain approve, an expired request and no round; `OpenRequest` records `Cap`; `SupersedeCapReject` succeeds at the cap and keeps `review.md`, and refuses with `ErrNotAtCap` below the cap, on a latest approve, on an open round with no verdict, and on a request with `Cap` 0.

**ImpactSummary:** Adds store fields and methods only; existing callers and on-disk rounds read unchanged, with the new keys absent.
