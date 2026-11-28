```yaml
lens: fix-code-correctness
outcome: reported
```

Task: correctness check of the fix's changed text (fix-code-correctness lens).

no findings

What I ran through:
- **Test deletion:** `measurementEntryRule` and both of its uses are gone. The gate run on `./internal/darnengine` and `./contracts/stencils` passed. The fix added no test.
- **No parent, no `Added after Discussion` entry:** the writer writes the stop report and Step 3's stop marker. `darnwrite.go:189-210` turns that into Stuck, with the report plus `darnStopWayForward` as the reason.
- **Resume after the operator records an entry:** `darnwrite.go:166-170` removes the old marker. The new spawn finds the allowing entry and runs the calls within its limits, so it does not stop again.
- **Refused post with no answering strand:** this takes the same stop path, which agrees with the parent directive's "your role's instructions send a stop".
- **Ungated delegation:** item 2's new line and item 3's "no subagent runs them" agree, and the "item 3" cross-reference points at the right item.

Outside my lens, not counted as a finding: a parent that answers "run none" still gets no stop path. The fix did not change that behaviour.

Tool calls:
1. Read `darnwiring.go` around the darn table and grepped for the stop marker's consumers, because my context lacked how the writer's gates are wired.
2. Read `darnDeps` in `darnwiring.go` and grepped the gate and seat packages for stop-marker handling, because my context did not show whether anything besides `darnwrite.go` reads the marker.
