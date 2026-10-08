---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/05-two-line-status-bar.md
    summary: Card 5 pins pane-border-status top, which breaks existing tmux-tier assertions on the row-0 pane (pane_top 0, pane height equal to CollapsedRows) in files no card lists or describes
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/08-title-row-geometry.md
    summary: Cards 8 and 9 append pins and zoom entries to the attach chain, but four attachgeometry integration tests hard-code a 10-element chain whose last element is the layout string, and no card says to change them
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/13-webster-review-segments.md
    summary: Cards 12 and 13 claim an existing spec test asserts the role beside the new Segment assertion, but that is false for four spec builders, and the affected packages sit at their test-budget ceiling with no budget edit in either card
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/03-strand-pane-options.md
    summary: Card 3 plans a new paneoptions_test.go table over the private strandLabel helper although the launch-fixture test it also extends already pins the label per strand name
---

### [MEDIUM] Card 5's title-row pin breaks tmux-tier tests that no card lists

**Location:** `_lyx/plan/05-two-line-status-bar.md` (the card that first pins `pane-border-status top`); affected tests `internal/reedcli/smoke_lifecycle_test.go:300`, `internal/reedengine/strandops_integration_test.go:79`, `internal/reedengine/attachgeometry_integration_test.go:245-251` and `:364-370`.
**Issue:** D7 records that with `pane-border-status top` the pane at window row 0 reports `pane_top` 1 and a content height one below its cell.
I confirmed it on tmux 3.6 with a private socket: a sole pane in a 24-row window reads `pane_top 1`, `pane_height 23`.
Four existing tmux-tier assertions sit on that row-0 pane:
- `smoke_lifecycle_test.go:300` requires the rebuilt Selvage pane (the session's sole pane) to read `pane_top` `0`.
- `strandops_integration_test.go:79` requires the replaced top strand's pane height to equal `cfg.CollapsedRows`.
- `attachgeometry_integration_test.go:249-251` and `:369-370` require the collapsed parent, `Strands[0]` and so row 0, to have height `cfg.CollapsedRows`.

Card 5 lists `attachgeometry_integration_test.go` only for its `rows-1` checks, and card 8 only for new D7 rows.
Neither lists `smoke_lifecycle_test.go` or `strandops_integration_test.go`.
`## verify:` compiles the `tmux` tier and never runs it, so the first run of these four is Publish, and the decision record's acceptance criterion "the `tmux`-tier tests pass" fails there.

**Fix:** In card 5, the first card that pins the title row, add `internal/reedcli/smoke_lifecycle_test.go` and `internal/reedengine/strandops_integration_test.go` to the Edit list.
State in the Intent that the row-0 assertions become the content-row value (planned cell minus the title row) and that the Selvage-only `pane_top` check expects `1`.
Say the same in card 8 for the `attachgeometry_integration_test.go` height checks, since card 8 is where the row-0 pin is adjusted.

### [MEDIUM] The attach chain's new shape breaks four integration tests that card 8 and card 9 do not mention

**Location:** `_lyx/plan/08-title-row-geometry.md` and `_lyx/plan/09-zoom-bracket.md`; affected `internal/reedengine/attachgeometry_integration_test.go:202,205`, `:296,299`, `:396` and `:489`.
**Issue:** Card 8 appends one `resize-pane` per pin after the chain's `select-layout`, and card 9 wraps the chain in a record-and-unzoom entry and a re-zoom entry.
The chain is then no longer ten argv elements, and its last element is no longer the layout string.
Four integration tests assert `len(argv) != 10`, and two of them take `argv[len(argv)-1]` as `wantLayout` or `plannedLayout` for a byte-for-byte `#{window_layout}` comparison.
Card 8's Intent for this file says only that it "gains the D7 acceptance rows"; card 9 does not list the file at all.
The untagged `attach_test.go` is covered, because both cards say it asserts the chain's order and it is in their Edit lists.
The tmux-tier file is the one with the hard-coded shape.

**Fix:** In card 8, and in card 9 if it is the later card to change the chain, add the file to the Edit list and state that the existing chain-shape checks stop assuming ten elements and take the layout string from the `select-layout` element instead of the last one.

### [MEDIUM] Cards 12 and 13 assert coverage that the existing tests do not provide, and the budget leaves no room to add it

**Location:** `_lyx/plan/12-loom-landing-segments.md` and `_lyx/plan/13-webster-review-segments.md` (Tests paragraphs); `internal/mergeresolve/mergeresolve_test.go:651-672`, `internal/websterengine/recoverbatch_test.go`, `internal/shedadapters/bouncer_judge_test.go`, `internal/treadleengine/judge_test.go`.
**Issue:** Both cards say each package's existing spec test "asserts the built spec's `Segment` beside the role it already asserts" and add no test function.
That holds for the driver, discussion, plan, rework, describe, friction, Merriam, burler and seed specs.
It does not hold for four builders:
- `mergeresolve`: the test over `buildConflictSpec` asserts `Prompt` and `Skills` only, never `Role`.
- `recoverbatch.go`'s recovery spec: `recoverbatch_test.go` runs a real `shuttleengine.Runner` over `shuttlefake` and never inspects the built spec.
- `bouncer.go`'s judge specs: `bouncer_judge_test.go` asserts `Prompt`, `Skills` and `OutputFiles`, not `Role`.
- `targeting.go`'s spec: its role is asserted in `treadleengine/engine_test.go:1078`, which card 13 does not list, and `judge_test.go` covers only the judge call.

`cmd/lyx/testdata/test-budget.yaml` pins `internal/mergeresolve` at 17 and `internal/websterengine` at 184, both equal to their current counts, and neither card edits the budget file.
An implementer who adds the missing assertion as a new test function therefore fails `TestTestBudget_PackagesStayWithinBudget`.
One who skips it leaves the Segment of those four specs unpinned.

**Fix:** Per spec builder that has no assertion today, name the existing test that is extended and the seam that exposes the built spec: for example the recorded `AddSpec` on `shuttlefake.Reed` for the recovery spec, or a new row in the existing table at `mergeresolve_test.go:651`.
Add `internal/treadleengine/engine_test.go` to card 13's Edit list for the targeting spec.
If a new top-level test function is unavoidable for any of them, add the budget file and that package's row to the card.

### [LOW] `strandLabel` gets its own table although the launch-fixture test already pins the label

**Location:** `_lyx/plan/03-strand-pane-options.md` (Tests paragraph, `paneoptions_test.go`).
**Issue:** `PATTERN-test-economy` asks for tests at the public surface and for no test per symbol.
Card 3 creates `paneoptions_test.go` with one table over the private helper `strandLabel`, and in the same card extends `TestLaunchStrandLocked_SplitWindowAndPaneTitleCalls` with launch fixtures that assert the `@strand` value after the title.
Those fixtures can carry the four names (prime, task, numbered role, unparseable) and so cover the same mapping through the real launch path.
The new file costs a budget unit and a second place to keep in step with `agentname.Parse`.

**Fix:** Drop `paneoptions_test.go` and have the extended launch-fixture test use the four name shapes; keep the file only for behavior of `setStrandPaneOptionsLocked` or `reassertStrandPaneOptionsLocked` that no existing test reaches.

## Prior rounds

Read after this review was saved.
Round 2's six fixes still hold in the current plan: card 15 lists `menu.go`, card 14 sets the `orchengine` budget row, card 5 lists both `rows-1` test files and the sandbox suite, card 6 lists `yamlengine.md`, card 2 keeps only the `segmentColor` table, and card 4's doc text no longer names an optional verb.
No regression found, and none of round 2's items is deferred or disputed.

## Focus verdicts

- **Caller completeness for `ideengine.Spawn`:** not a defect. Every caller of `Spawn` is in card 15's Edit list: `internal/ideengine/menu.go`, `internal/idecli/cli.go`, `spawn_clean_integration_test.go` and `spawn_hub_integration_test.go`. `spawn_scenario_integration_test.go` and `menu_test.go` reach it through the listed helpers and `Menu`. `SpawnDriven` is a separate function and unchanged. `cmd/lyx/main_test.go`'s `ide spawn` row fails in the pre-run before it reaches `Spawn`.
- **Test budget:** cards 1-11, 14 and 15 that add or remove top-level test functions list `cmd/lyx/testdata/test-budget.yaml`. Card 14 lists it, names `watcher_test.go` and states the new watcher behaviors are new functions. The one gap is F3, for cards 12 and 13.
- **Status bar height:** every `rows-1`, `finalRows-1`, `resizedRows-1` and 41-of-42 window-height assertion is in card 5 (`watchdog_integration_test.go:153,221,255`; `attachgeometry_integration_test.go:218,427,519`). The `reedcli` watchdog and dot-fill tests use `resize-window` on detached sessions and do not depend on the status rows. The neighbouring row-0 and chain-shape breakages are F1 and F2.
- **Retired text:** not a defect. `docs/shared-libs/yamlengine.md`, `tools/sandbox/SANDBOX-REED-SUITE.md`, `configsync_test.go`, `configreg/fill_test.go`, `.gitattributes`, `PATTERN.md` and the `pattern/` files are in card 5 or card 6. The remaining hits are generic YAML fixtures in `internal/yamlengine/fill_test.go`, generic tmux reference text in `docs/reference/tmux_scripting.md`, and one historical comment in `internal/reedcli/smoke_dotfill_test.go:123`; none asserts or cites the retired reed key.
