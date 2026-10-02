# Card 2 — one-dispatch-scope

**Edit:**
- `internal/websterengine/beginbatch.go`
- `internal/websterengine/runlevel.go`
- `internal/webstercli/validate.go`
- `internal/websterengine/begun_test.go`
- `internal/websterengine/beginbatch_test.go`
- `internal/websterengine/runlevel_test.go`
- `internal/webstercli/verbs_test.go`

**Prosa:**
- `internal/websterengine/doc.go`

**Uses:**
- `internal/planglyph#ValidateDispatch`

**Intent:**
Compute dispatch scoping once and use it at `begin-batch`, `run` entry and `lyx webster validate` (#329, decisions D1 and D2).

Add an exported `DispatchScope(batches []batcher.Batch, st *State) (begun, forthcoming []planparser.Card)` to `beginbatch.go`.
`begun` is what `begunCards` returns today (every card of a batch begin-batch recorded, terminal or not);
`forthcoming` is the cards of every begun batch whose record is not terminal.
A terminal batch's cards are never forthcoming: its done-checks proved its targets present.
A nil state returns two nil slices.
`BeginBatch` and `Run` call it and pass both sets to `planglyph.ValidateDispatch`; their comments name the forthcoming half.

In webstercli's `validate.go`, delete the method `completedCards` and its terminal-only walk.
`scopedValidate` takes the state `validateCmd` already loaded under the lease (no second `LoadState`), sequences the batches as today, and calls `websterengine.DispatchScope`.
With no begun card it runs `planglyph.Validate` (scope `whole-plan`, approval gate included);
with any begun card it runs `planglyph.ValidateDispatch(plan, root, begun, forthcoming)` (scope `pending`).
A nil batcher with a state on disk stays the `ErrNilBatcher` wiring error.
Update the file doc comment, `scopedValidate`'s comment and `validateCmd`'s `Long` help: `pending` now starts once a run has begun any batch, and lists which cards are excluded and which targets count as forthcoming.

Bound (D2): `validate` stops resolving begun, non-terminal cards, so drift in such a card's own targets is reported by none of the three sites.
That card was fully validated at its first `begin-batch`, its bytes stay pinned by its recorded `CardHashes`, and `record-batch`/`recover-batch` still run its done-checks and drift detection.
A forthcoming target that never lands keeps the run from outcome done, because the run-exit check requires a terminal done record for every batch.

Rewrite `doc.go`'s dispatch-scoping text (the "every terminal batch runs the same mechanical pass" and staleness sections where they describe what begin-batch and run re-resolve) to state the one rule and the forthcoming set.

Tests:
- `begun_test.go`: `DispatchScope` over a terminal batch, a begun non-terminal batch and an unbegun batch returns the first two in `begun` and only the second in `forthcoming`; a nil state returns nil for both.
- `beginbatch_test.go`, #329 Seen: card 2 creates a file, cards 3 and 6 Use it; `BeginBatch` for batch 2 records it, no commit lands, and a second `BeginBatch` for batch 2 succeeds with no blocking finding.
- `beginbatch_test.go`: a re-begun batch whose fork committed its Create target re-begins without `create-already-exists`.
- `runlevel_test.go`: `Run` entry on the Seen state passes validation and reaches the Master spawn.
- `verbs_test.go`: `lyx webster validate` on the Seen state exits 0 with scope `pending`; update `TestValidateCmd_MidRunScopesToPendingCards` for the begun-based switch.

**ImpactSummary:** begin-batch, run entry and validate change which plan refs they resolve on a run with begun, non-terminal batches; validate's `pending` scope now starts at the first begun batch.
