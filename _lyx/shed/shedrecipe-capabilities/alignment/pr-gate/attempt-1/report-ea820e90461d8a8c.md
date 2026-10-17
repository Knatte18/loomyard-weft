---
verdict: APPROVED
---

## Alignment at pr-gate: shedrecipe-capabilities, PR 719 at 076d0cd58

Measured: the diff `main...076d0cd5897e4af2d66cd1324f175c04c9d6d4e2` (merge base `649fe0b4c`, equal to `main`) against board entry `shedrecipe-capabilities` (operator decision 2026-10-17, steps 1 and 2) and decisions D1 to D8 of the decision record.

### Goal and decisions

- Step 1, per-segment settings: `Env.Segments map[string]SegmentSettings` replaces `RowReviewModels`, `RowModelKeys`, `RowMaxBounces`, `ReviewMaxBounces`, `RowHardMaxBounces`, `RowBouncerModels`, `DarnReviewMaxBounces` and `ReviewModels`, all gone from `internal/shedrecipe/recipe.go` (D1).
  `SegmentSettings` carries the models, their keys, the Bouncer pair, the budget, the hard cap, their keys and the config file, as D1 lists.
- Step 2, one told lookup: `Env.Setting func(module, key string) (any, error)` names no `shedrecipe` type (D4); `configreg.SettingLookup` resolves through strict `configengine.Load`, checks keys against the embedded template, and snapshots each module once.
  `configreg` joins the strict-caller list in `cmd/lyx/configstrictness_test.go` and `pattern/PATTERN-config-strictness.md`, as the constraints require.
- D2: `Constructor` takes `RowIdentity{Name, Segment}`, filled by `shedbuild.Build`.
- D3: `rowSegment` in `internal/shedrecipe/entries_settings.go` refuses a row with no segment or no `Segments` entry, with a way forward.
- D5 and D6: the loom and darn recipes name `reviewer_fan_setting`, `fixer_fan_setting` and `auto_accept_setting` as D5 lists; the darn recipe names no auto-accept setting; malformed references and a nil `Setting` are refused at construction.
- D7: `TestShippedRecipes_FanSettingsAgreeWithinSegment` in `internal/loomrecipe/fansettings_test.go` checks reference agreement and both presence agreements, which settles the Discussion-Review carry-over `fan-setting-presence-agreement-unpinned`.
- D8: `loomSegmentPerches` in `internal/loomcli/wiring.go` is the segment-to-perch table, and the darn wiring takes its keys from `internal/reviewbudget`.
- Out-of-scope items stay out: no `internal/battencli` change, no `loom.yaml` or `darn.yaml` template key change, and no row entry gate or `Command` producer work.

### Carried-over risk: Webster-Review `unrelated-pattern-entries-rewritten`

Round 1's circling decision accepted on budget with this BLOCKING finding open.
The fixer restored two qualifiers and left eight entries shorter (`PATTERN-mutation-record`, `PATTERN-plan-generation`, `PATTERN-gate-self-check-parity`, `PATTERN-creel-store`, `PATTERN-friction-capture`, `PATTERN-wait-mark-display-only`, `PATTERN-spawn-observability`, `PATTERN-documentation-lifecycle`), because the full restore put `PATTERN.md` 264 bytes over the overview headroom line.
I judge this not a scope deviation:

- The recorded PATTERN change, the new `PATTERN-recipe-setting-lookup` line, cannot land without cutting bytes, and `internal/patternlint`'s way forward prescribes it: "cut the named bytes from PATTERN.md, moving detail into the entries' background files".
- Every dropped qualifier is already in its unchanged background file: the envelope keys (`pattern/PATTERN-mutation-record.md:3`), `round-<N>/prior-generation/` (`PATTERN-plan-generation.md:5`), "in the same task" (`PATTERN-gate-self-check-parity.md:24`), archiving only after a reflection (`PATTERN-friction-capture.md:3,10`), "No Go decision reads them" (`PATTERN-wait-mark-display-only.md:6`), "real" process and teardown logging (`PATTERN-spawn-observability.md:3`), the sole writer and `Delivery` seam (`PATTERN-creel-store.md:3`), and no design doc for unbuilt work (`PATTERN-documentation-lifecycle.md:3`).
- No invariant changes meaning; only the overview lines get shorter.

### Overlap with peers and parent commits

`lyx loom peers` lists no parent commits and one peer.

- `creel-notices` (Plan-Write, row 7; this run's `own_row` is 22): it lists no changed paths yet.
  Its decision record changes `PATTERN-orch-pane-single-writer` and `PATTERN-durable-vs-ephemeral-state`, neither of which this run touches.
  It names `PATTERN-creel-store` only as a constraint, and this run shortens that entry's overview line with its meaning unchanged.
  It plans to remove `NoticeLog.Now` fields from `internal/testkit/envkit/envkit.go`, while this run changes the `Segments` and `Setting` fill in the same file.
  Both overlaps are text-only: the same files, changed in different places.
  Landing order: this run (row 22) is further along and lands first, and `creel-notices` takes its changes at Merge-In.

Shared bookkeeping, no escalation: `cmd/lyx/testdata/test-budget.yaml` (`internal/configreg` 10 to 11, `internal/loomrecipe` 56 to 57) and the new `PATTERN-recipe-setting-lookup` index line.

### Superseded

No other board entry is made obsolete; `row-entry-gate`, `command-producer` and `config-migrate-hub-layer` stay open, as the scope's Out list says.
