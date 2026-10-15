# Fixer report — merge, round 1

## Changes

- F1 (fixed as proposed): recordless `Send` and `Interrupt` now resolve the engine from the live strand's provider tag.
  - `reedengine.StrandStatus` gains `Provider`, filled from `Strand.Provider` in the status listing.
  - `liveStrand` in `run.go` returns the matching strand; `requireLiveStrand` wraps it.
  - `Interrupt` uses `RunState{Engine: strand.Provider}`; `recordlessSendContext` takes the provider and does the same. An empty tag still means the default engine.
  - The `Send_and_Interrupt_on_a_live_strand_with_no_record…` test in `enginedispatch_test.go` is renamed and gains rows for a strand tagged with the non-default engine.
  - Verified with `lyx gate test ./internal/shuttleengine ./internal/reedengine`: both pass.

- Gate repair: `PATTERN.md` overview was 84 bytes past its headroom line.
  Shortened the `PATTERN-pair-teardown` and `PATTERN-model-choice-mapping` entries (the pair-teardown detail is already in its background file); the overview is now 15563 bytes.
  `lyx gate test ./internal/pattern ./internal/shuttleengine` passes.

- Gate repair: the `recordlessSendContext` doc in `send.go` wrapped a sentence across two lines; it is now one sentence per line.

## Disputed

None.

## Deferred

None.
