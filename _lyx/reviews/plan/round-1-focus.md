---
round: 1
focus:
  - "Card 1's Intent order against D1-D5: does every refusal the decision record names (both dirtiness checks and both status probes) sit ahead of archiveWeftTip, and does the archive sit ahead of removePortal/removeLaunchers, with force still skipping only the two dirtiness checks?"
  - "Card 1's test rewrites against D6 and the Acceptance criteria: does the card cover each listed acceptance case (weft-dirty refusal, warp-dirty refusal, resume-archives-once, same-tip reuse, warp-probe failure)? Does each case assert what the criterion requires, for example the weft-dirty case's portal, launchers, both worktrees, both branches and empty mutation record?"
  - "Scope licence: does anything in Card 1 go beyond the decision record's Scope In list? Candidates to settle are the livestate_doc_test.go anomaly-section deletion and the new TestRemove_WarpStatusProbeFailurePushesNoTag. Does anything the record puts Out of scope (cleanup.go, archive.go, battencli teardownRefusal, pruning old tags) get touched?"
  - "Other old-order dependants: livestate_verbs_test.go's header (lines ~24-26, ~361), livestate_refusal_selftest_test.go, livestate_refusal_test.go, livestate_states_test.go and remove_siblingdirty_integration_test.go all reference Remove's dirty pre-flight. Does the card name every file whose assertions or doc comments pin the teardown-before-refusal order, rather than only the ones D6 lists? Leave aside line-number citations that were already stale before this change."
  - "Verify coverage: the verify section runs integration tests for fabricengine, battencli and fabriccli. Does any package Card 1 targets go unrun? Check this against the decision record's open risk that other integration tests drive a dirty no-force Remove (ideengine's spawn_hub_integration_test.go calls Topology.Remove)."
  - "ImpactSummary: does it state a real blast-radius conclusion (callers of Topology.Remove across packages, what changes for them), or does it restate Intent?"
  - "Granularity: is one card justified by the overview's claim that the reorder breaks every old-order test at once, or does the remove.go production change separate into an independently reviewable unit?"
---

Single-card plan reordering `Topology.Remove`.
The main risks are fidelity to D1-D7 and the acceptance list, and completeness of the old-order test and comment sweep across the `livestate_*` harness files.
Dependency-edge completeness and the mechanical check set are excluded per the rubric.
