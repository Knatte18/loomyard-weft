# Pilot one coverage map: build time, full-set estimate, and whether the map is usable

`test-redundancy-candidates` · Task · enhancement, testkit, gate · running · Darn-Write

## Idea (operator, 2026-11-11, narrowed 2026-11-12)

The coverage maps exist to cut the test suite, which has far too many tests for this repository.
Before the full set is built, a pilot on one package answers three questions in order:
can a map be built, what does it cost, and is the map of any use at all.
The task stops after the pilot's report; the full set, and any cut, wait for the operator.

## What it does

1. Build: pick one package with tier 0 or tier 1 tests of a middling count, not `internal/fabricengine` (swept by hand on 2026-11-08 and being reworked).
   State the choice and why.
   Build its map by a hand call of `lym gate testmap` naming the package, once.
   Record whether it succeeded, the wall time, the CPU time and the gate slots it held; a failure is reported with its cause and not rerun.
2. Estimate the full set: from that one build, estimate the wall time, CPU time and gate-slot time to build every package that has tier 0 or tier 1 tests, with the package list and each package's test count as the scaling input.
   Give the estimate as a range with the assumption that makes it a range, and say how long the upkeep dobby's passes would take to drain it.
3. Use: check whether the map is of any use, on that package only.
   - Redundancy: produce the candidate list (every test whose covered blocks the package's other tests already cover, with its run time) by the judgment of `internal/testcover/verdict.go`.
     Tests marked `//testtiming:keep` and spawn-marked tests are neither candidates nor covers.
     Read at most five candidates against the tests that cover them and say, for each, whether its assertions add anything the covers lack.
     If no verb gives the candidate list today, add the smallest one for it, with its tests.
   - Scoping: take at most two landed commits that touched the package and compare the tests the selector names for each diff with the tests that actually cover the changed code.
4. Verdict: one short section saying whether the map is usable for cutting, for scoping the gate, for both or for neither, and what a full run would then cost.

Write the report under `docs/research/speed/`.

## Cap

One package, one build, one candidate list, at most five candidates read and two commits compared; no repeat runs.

## Not in scope

Building any other package's map, cutting or folding any test, and any change to the maps' format or upkeep rules.
