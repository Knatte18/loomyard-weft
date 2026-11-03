```yaml
lens: fix-way-forward
outcome: reported
```

Task: check, through the fix-way-forward lens, that every way forward the fixer added can be carried out from the state it names and makes progress.

Introduced ways forward checked:
- D5 Bound: "until the healing loom start replaces it". An old-key status strand's sidecar names the old build, so `StrandCurrent` fails and the new-build loom start replaces it under D3 with key 1, placing it after the runs strand. This is reachable and makes progress.
- D5 Bound: "a dropped runs strand comes back at the next `lym beam start` under the new build (D4)". With no runs strand, the new `ensureRunsStrand` adds one at key zero beside the status strand. The old driver's Beam-Up row runs once, so it does not drop the strand again on every pass. This is reachable and makes progress.
- D5 Bound: "a replaced status strand at the next loom start (D3)". D5 already says a start that finds no status strand adds it. That start comes from a batten pass or a hand `lym loom start`, which is re-entrant. The runs strand that the old driver put in place carries an old-build sidecar, so the next `lym beam start` replaces it. This is reachable.
- Acceptance criterion D2, "a viewer on the lower-index overview window that ends moves to the other overview window": `RemoveStrand` or reconcile can end that window, and they reach `selectRelevantWindowIfCurrentLocked` with an ending set. A test can drive it. An overview-for-overview replacement selects its new window instead under the Constraint, so the criterion does not conflict with that path.

no findings

Tool calls: none; my context already held everything needed.
