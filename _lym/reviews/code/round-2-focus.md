---
round: 2
focus:
  - "internal/loomcli/wiring_darn_test.go and internal/loomcli/darnwiring.go: does a test wire a darn recipe and call the DarnTable closure with a pair slug and a non-empty darn.gate_fixer, with an empty darn.gate_fixer, and with no pair slug, and assert the gated no-gate Step 4 in the first case and the ungated Step 4 in the other two? Would a regression of the GateFixed choice fail a test?"
  - "internal/darnengine/spec_test.go TestDarnSpec_Refusals: does its comment match what DarnTable reads, and do rows remove the make_change stencil and the gated make_change stencil (with GateFixed set) and expect an error naming the removed stencil?"
  - "contracts/stencils/markers_test.go: does the position table anchor the make_change marker under the Step 4 heading of the darn-template-write stencil?"
  - "internal/darnengine/spec_test.go TestDarnSpec gated row: do its assertions pin only what tells the gated and ungated Step 4 apart, with no extra assertions on the fragment's static prose?"
---

Round 2 re-checks the four round-1 finding sites after the fixes in the range's later commits.
