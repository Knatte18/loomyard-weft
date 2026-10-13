# Shuttle turn-in-flight veto and twill lens-review gate

Both batches recorded `done`, with no recovery and no warnings.

## Batch 01-shuttle-turn-in-flight-veto

Head `673469b07073f1a9f23940ab982b2352936c50ff`.
Cards 1 to 3 landed.
An unmatched turn start that the engine reads as in flight now vetoes the idle-pane override, and a later session start lifts the veto.
Claude reads a turn's progress from the newest main-chain transcript entry, pinned by `TestTurnStartProgress` over fixtures under `testdata/turnprogress/`.
The run record carries fork authorization from spawn, and the fork audit and a gate entry's `Forked` closure read it.
The scope guard listed one deviation: `internal/shuttleengine/sessionsignal.go`.

## Batch 04-loomshed-lens-review-gate

Head `b61695ae9de0665d426d8c62fe67a1ccb152f1d6`.
Cards 4 to 7 landed.
`loomshed` gained `ValidatePlanLensReview` and its gate, which check that `lens-review.md` is present.
Only Plan-Write's plan gate entry carries the lens-review closure.
`lyx loom validate-plan --lens-review` is the matching self-check mode, with flag refusals and a parity entry.
The twill directive's step 8 takes its own command: the Plan host passes `--lens-review`, and the Discussion host's prompt is unchanged.
The scope guard listed one deviation: `internal/loomcli/parity_test.go`.
