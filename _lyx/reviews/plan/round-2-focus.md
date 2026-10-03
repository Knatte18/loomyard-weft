---
round: 2
focus:
  - "Card 19 (locationkit-callers) footprint: card 19 deletes newPortalLauncherTestLocation from internal/fabricengine/portallauncher_test.go. At HEAD it is also called from internal/fabricengine/hubworkspace_test.go, launcher_content_test.go and origin_test.go, and origin_test.go's header comment names it. Settle whether card 19's Edit list and ImpactSummary now cover every one of those files, so the package still compiles after the card."
  - "Card 44 (slices-cleanup) footprint: card 44 replaces equalStringSlices, defined in internal/reedengine/reconcile_test.go, with slices.Equal. At HEAD it is also called from internal/reedengine/strand_test.go and lifecycle_test.go. Settle whether card 44's Edit list and ImpactSummary cover both call sites."
  - "Fidelity to the acceptance criterion that no consumer package keeps its own Shuttle or merge-shuttle fake except the cycle-forced cases: internal/mergeresolve/mergeresolve_test.go and internal/frictionengine/reflect_test.go each define a Run-only fakeShuttle with per-call results and errors. No card converts either, and none of shedfake's imports (shuttleengine, burlerengine, shedengine, websterengine) depends on mergeresolve or frictionengine. Settle whether the decision record's D1 and acceptance criteria require these fakes to move to shedfake. If they do, settle which card carries each conversion. If they do not, settle where the plan records why they stay."
---

Every round-1 finding was confirmed fixed, so this focus covers only what remains open or unsettled.
A `grep -rlw` caller sweep already confirmed the target lists of cards 4, 5, 7, 9, 21, 23, 26–28, 30, 32, 33, 38, 40 and 43.
