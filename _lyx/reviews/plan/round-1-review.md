---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/08-batten-notice-log.md
    summary: Card 8 adds a nil-defaulting Now func inside NoticeLog on InnerRunDeps and RowNoticeDeps but never registers the new nested seam paths in envkit's nilLegal, so envkit.NilSeams reports them and the seam-completeness tests fail
    fix: "In card 8, add `internal/testkit/envkit#nilLegal` to **Edit:** and state in **Intent:** that it gains the three paths `InnerRun.NoticeLog.Now`, `Land.Notices.NoticeLog.Now` and `Teardown.Notices.NoticeLog.Now`, each nil-legal because a nil Now reads time.Now (the same reason `InnerRun.Now` and `Land.Notices.Sleep` are listed there). Add a line to the ImpactSummary naming envkit's nil-legal list. No test-budget change."
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/01-render-size-vocabulary.md
    summary: Card 1 says layout.go is untouched, but it edits render#placement, which is declared in layout.go, and its Intent rewrites placement.strip's comment
    fix: "In card 1's **Intent:** first paragraph, replace the parenthetical `layout.go`/`checksum.go` are untouched with: no mechanics in `layout.go` or `checksum.go` change; `layout.go` changes only the comment of `placement.strip`, which now names every collapsed positional cell and every content cell. Keep `internal/reedengine/render#placement` on **Edit:**."
  - id: F3
    severity: LOW
    class: design
    location: _lyx/plan/09-batten-overview.md
    summary: Cards 7 and 9 each plan their own copy of the whole-minute duration format (12m, 1h05m, 2d03h) that the minute-resolution-times shared decision names for both panes, so two packages carry the same formatter
    fix: "In card 5, add a **Create:** bullet `plan:internal/statuspane#FormatMinutes` -> `func FormatMinutes(d time.Duration) string`, which renders a duration in whole minutes as `12m`, `1h05m` or `2d03h`, with no new test of its own: the exact time strings `TestRenderChecklist` and `TestRenderOverview` already assert cover it. In card 7's renderChecklist bullet and card 9's renderOverview bullet, replace the inline format description with a call to `statuspane.FormatMinutes`, and add `plan:internal/statuspane#FormatMinutes` to each card's **Uses:**."
  - id: F4
    severity: LOW
    class: design
    location: _lyx/plan/09-batten-overview.md
    summary: Card 9 plans TestFitOverview as its own top-level test for the fitOverview helper, a one-test-per-symbol split, while card 7 tests the matching checklist Fit as rows of its renderer's table test
    fix: "In card 9's **Tests:**, drop `TestFitOverview` and move its two cases (three lines into two rows keep the oldest line and a summary naming one run not shown; lines that fit come back unchanged) into `TestRenderOverview` as rows that pass `renderOverview`'s output through `fitOverview` with a told row count. Lower the `test-budget.yaml` bullet to `internal/battencli` 69 → 72."
---

### [BLOCKING] Card 8's new NoticeLog.Now seam breaks envkit's nil-seam checks

**Location:** `_lyx/plan/08-batten-notice-log.md`

**Issue:** Card 8 gives `battenshed.NoticeLog` a `Now func() time.Time` field ("nil reads `time.Now`"), and adds a `NoticeLog NoticeLog` field to both `RowNoticeDeps` and `InnerRunDeps`.
`shedrecipe.Env` carries `InnerRun battenshed.InnerRunDeps` (`internal/shedrecipe/recipe.go:259`), and `RowNoticeDeps` sits in it as `Land.Notices` and `Teardown.Notices` (`internal/battenshed/land.go:83`, `internal/battenshed/deps.go:317`).
`envkit.NilSeams` (`internal/testkit/envkit/envkit.go:334`) recurses into nested structs and reports every nil func field not in `nilLegal`.
`nilLegal` already lists the nil-defaulting fields of these same structs (`InnerRun.Now`, `Land.Notices.Sleep`, `Teardown.Notices.Sleep` at `envkit.go:70-81`), but card 8 does not add the three new paths.
Card 8's `battencli.noticeLog` sets only `Path` and `AttachCommand`, so after `wire` the paths `InnerRun.NoticeLog.Now`, `Land.Notices.NoticeLog.Now` and `Teardown.Notices.NoticeLog.Now` are nil.
That fails `TestWire_EverySeamFilled` (`internal/battencli/wire_seams_test.go:78`), which is in card 8's own package and so fails under the card's own gate.
It also fails envkit's own `NilSeams(FullEnv(t))` assertion (`internal/testkit/envkit/envkit_test.go:12`) and loomcli's seam tests (`internal/loomcli/wiring_seams_test.go:79`, `wiring_darn_test.go:52`), which list the batten-only notice seams by path.
None of these files or symbols is on card 8's targets.

### [LOW] Card 1 says layout.go is untouched but edits a type declared there

**Location:** `_lyx/plan/01-render-size-vocabulary.md`

**Issue:** The Intent opens with "`layout.go`/`checksum.go` are untouched", but `internal/reedengine/render#placement` is on **Edit:**, and the Intent rewrites its `strip` comment ("`placement.strip` is true for every collapsed positional cell and every content cell ... its comment ... say which cells pin").
`placement` is declared in `internal/reedengine/render/layout.go:20`, and its `strip` comment today says the field marks "a collapsed placement".
The comment edit is correct and needed, and no mechanics change, so `PATTERN-leaf-packages` and the decision record's render constraint hold.
But a builder reading the card literally gets two conflicting instructions about `layout.go`.

### [LOW] The minute-resolution duration format is planned twice

**Location:** `_lyx/plan/09-batten-overview.md`

**Issue:** The overview's Shared Decision `minute-resolution-times` puts both panes on one time format.
Card 7's `renderChecklist` bullet ("rendered in minutes as `12m`, `1h05m` or `2d03h`") and card 9's `renderOverview` bullet ("times render in minutes as `12m`, `1h05m` or `2d03h`") each describe the same formatter inline in their own package, `loomcli` and `battencli`, with no shared symbol.
Card 5 builds `internal/statuspane` as "the one implementation of the pane mechanics the runs overview (card 9) and the gate checklist (card 7) share", which is the natural home for a formatter that two call sites need.

### [LOW] TestFitOverview is a per-symbol test

**Location:** `_lyx/plan/09-batten-overview.md`

**Issue:** Card 9 plans `TestRenderOverview`, `TestFitOverview` and `TestOverviewSource_ReadsThePrimesRunsFromDisk`, one top-level test for each of `renderOverview`, `fitOverview` and `overviewSource`.
`fitOverview` is a helper serving the overview frame alone, and its two cases are inputs and outputs of the same rendering behaviour `TestRenderOverview` pins.
`PATTERN-test-economy` makes such cases rows of the existing behaviour's test.
Card 7 does exactly that for the checklist: its `Fit` cases are rows of `TestRenderChecklist`.

## Notes on the focus directives

Every directive was reached; none was departed from.

- D3 (cards 1 and 2): the `top` refusal for positional and sharing kinds is in card 1's `ValidateDisplay` (reached by `TestFixedHeightPinsMatchesRulesPlacedHeights`' `wantErr` rows) and card 2's `validateDisplay` at both op boundaries (reached by `TestAddStrandLocked` and `TestUpdateStrandLocked` rows).
  The reclaim order is card 1's `clampToFit` by `cellRole`, pinned by the two short-box `TestRulesGolden` rows, and the bottom-most-strand case is the sole-content-strand row.
  No card adds scope outside the decision record's Scope or Out.
- D4 (card 3): `rows` 0, an unknown name, a positional strand's name and a wrong pane id each have a named input and a `TestSetContentRows` row, and the unchanged count has its own row.
- D7 and D8 (cards 7 and 8): every time rule (folded repeats, `goto`, unrecorded calls, removed rows, unreadable recipe) is a `TestRenderChecklist` row.
  Every D8 rule (once per process at the first attempt, one `O_APPEND` write, told path, logged failure) is carried by card 8.
  The cleanup-rebase constraint is placed in card 8, and the branch HEAD is `bc9ee7a02`.
  Each doc the decision record names lands in the card that changes its code.
- Attack surface: cards 3, 5, 7, 9, 10 and 12 each state a bound in **Intent:**.
  Card 4 bounds the new bindings to the window carrying `@lyx_strands`, with a display-only marker.
- Card order: no card uses a symbol a later card creates.
  Card 6 moves the build identity into `statuspane` before card 7 and card 10 use it, and card 7 never touches loom's removed copy.
- Edit targets: the re-signed members carry arrows (`clampToFit`, `toRenderInputs`, `planLayout`, `fixedHeightPins`, `contentPinsLocked`, `planPaneTarget`, `resolveStatusStrandAction`, `attachLine`).
  Every caller of the re-signed reedengine methods, including test callers, is on card 2.
- Verify: `## verify:` runs `go build`, `go vet` under each tag, `go test ./...` and `go test -tags integration ./...` in the template's order, covering every targeted package.
  The test-budget arithmetic of each card matches the current rows and test counts.
