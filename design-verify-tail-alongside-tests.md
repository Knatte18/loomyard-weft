# Run a verify gate's reports alongside its tests

`verify-tail-alongside-tests` · Task · enhancement, gate, darn · running · Drift-Expect

## Seen

`darn-slowdown-measure` (`docs/research/darn-slowdown-measure.md`) found that each verify gate spends about 75 to 105 s after its tests:
the reach, redundancy and mutation reports and the red-first step run one after another once the tiers finish.
Over a darn's verify gates that is 4 to 7 % of a 45 min run.

## Goal

Run the three reports concurrently with the tier 2 tests, or only on the last verify gate before a handoff, whichever the code supports cleanly; keep red-first where it is.

## Bounds

Time one verify gate of the same diff before and after the change, once each, and state both wall times in the change description.
