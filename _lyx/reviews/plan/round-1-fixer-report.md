# Plan-Review round 1 — fixer report

## Fixed

- **F1 (MEDIUM), the driver rename strands pre-task `loom-driver` strands:**
  - Card 8 adds `loomengine.LegacyLoomDriverStrandName = "loom-driver"`. Its doc says why it exists.
  - Both driver lookups, `resolveDriverStrandAction` and batten's `driverAliveFrom`, also match that exact literal.
    A live pre-task driver therefore counts as live, so no second driver spawns, and a dead one is still found and removed by GUID.
  - New tests cover the legacy driver cases.
  - The overview's Shared Decision on `agentname.Matches` now states the driver exception and its reason.
- **F2 (MEDIUM), the standalone acceptance criterion had no carrying card:**
  - Card 10 creates `internal/standalonegeom/naming_integration_test.go`.
    It builds the reed engine from `standalonestate.Derive` and `standalonegeom.ReedGeometry` on a plain checkout with no `.lyx-code`, as the webster and burler standalone wiring does.
    It asserts the `<StandaloneCode(hash8)>:worker` name in state and pane title, `LYX_STRAND_NAME` set, and `LYX_PARENT` unset.
  - The card's Uses gains `standalonegeom/reedgeom.go` and `standalonestate/standalonestate.go`.
- **F3 (LOW), card 8's ImpactSummary:** now concludes on the blast radius: four lookups in three packages, pre-task `loom-status`/`orch` reachable by equality, and pre-task `loom-driver` reachable only through the legacy constant.
- **F4 (LOW), a malformed recorded code's way forward was a dead end:**
  - Card 2's `ReadCode` and the clone probe now treat a value failing `agentname.ValidateCode` as not found and log a warning, so `lyx fabric code <code>` and `clone --code` record over it. Card 2's test list gains the malformed-record case.
  - Card 7's refusal is now only the empty code, whose way forward works.
    A told malformed code is a wiring guard.
- **F5 (LOW), `Resolve`'s refusal named no way forward:** card 1's refusal now ends with `way forward: pass the role segment alone, or run the command from the worktree the full name belongs to`, and its test asserts the text.
- **F6 (LOW), card 11's ImpactSummary restated Intent:**
  - It now concludes that only reed-launched panes run the lines, after reed's export, so no line expands to an empty name.
  - It also says a resume line stored before the deploy lacks the flag until the watchdog repairs the name.

The `approved:` key in `00-overview.md` was not touched.

## Deferred

None.
