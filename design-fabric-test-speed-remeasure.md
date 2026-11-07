# Re-measure tier 1 and internal/fabricengine after the fabric test speed tasks land

`fabric-test-speed-remeasure` · Task · research, fabric, testkit, git · running · PR-Review

Decision record: [Sketch](decision-fabric-test-speed-remeasure.md)

Filed from the Sketch `fabric-test-speed`.

Takes: every other part landed on main, and the report's W1 figures of 75.0 s and 57.0 s
Delivers: one lym gate test --tier 1 --all -- -count=1 pass and one lym gate test --tier 1 ./internal/fabricengine -- -count=1 pass on a quiet neve, read against W1 and the expected 36 s, 56 s and 48 s, filed as a board note naming the two commands, the expected times and whether the hub's tiers carry {test_targets}
Exports: nothing

The part changes these packages:

- docs/research/speed (about 60 changed lines)
