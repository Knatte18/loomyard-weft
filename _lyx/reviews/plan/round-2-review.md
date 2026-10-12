---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/09-batten-overview.md:42
    summary: Card 9 builds the runs overview but puts its design in no package doc; battencli's package comment in cli.go, which names its other non-shedverbs verbs, is left without the overview verb, against PATTERN-documentation-lifecycle
    fix: In card 9's Intent, beside the docs/overview.md sentence, add that the battencli package comment at the head of cli.go (the file holding the already-targeted internal/battencli#Command) gains a paragraph on lyx batten overview, the hidden pane verb the orch start runs in the runs strand. The paragraph states which runs show and what one line holds (D2), the ordering, the poll and backoff, the two-strike unreadable rule, and the bound that its one write is the content-rows report reed confines to the runs strand. Add no new target, since cli.go beside internal/battencli#Command would trip redundant-file-target.
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/05-statuspane-pane.md:47
    summary: Card 5 introduces FormatMinutes with no test, deferring coverage to cards 7 and 9, whose named test rows include no duration of an hour or more, so the 1h05m and 2d03h forms are pinned by no test
    fix: "In card 5, replace the sentence It has no test of its own... with a new test: declare `plan:internal/statuspane#TestFormatMinutes` -> `func TestFormatMinutes(t *testing.T)` under Create, in pane_test.go. Make it table-driven with exact-string rows for a sub-minute duration, 12m, 59m, 60m (1h00m), 1h05m, 23h59m, 24h (1d00h) and 2d03h, so each format boundary is pinned. Change card 5's budget line to internal/statuspane: 3 and card 6's to internal/statuspane 3 → 6."
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/01-render-size-vocabulary.md:46
    summary: Card 1 prescribes ValidateDisplay refusals for an unknown SizeKind, ShareNum below 0, ShareNum above ShareDen and MinRows below 1, plus a hidden-display exemption, but names no test row that reaches any of them; only top positional, top sharing and ShareDen 0 are reached
    fix: In card 1's Tests, extend the TestFixedHeightPinsMatchesRulesPlacedHeights bullet with four more wantErr rows, each a below-parent strand where Rules errors and the pins are nil. The rows are Size{Kind - an unknown value such as fixed}, a content size with ShareNum -1 / ShareDen 3 / MinRows 3, a content size with ShareNum 2 / ShareDen 1 / MinRows 3, and a content size with ShareNum 1 / ShareDen 3 / MinRows 0. Add one non-error row where a hidden strand carrying a content size with ShareDen 0 lays out and pins exactly as the same window without that strand.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/01-render-size-vocabulary.md:67
    summary: Card 1's TestPartitionByAnchor row names a hidden top content strand, a display no Strand can carry since Display.Anchor holds one value, so the row cannot be written as stated
    fix: In card 1's TestPartitionByAnchor bullet, replace and a hidden top content strand is dropped with and a hidden strand carrying a content size is dropped.
---

### [BLOCKING] Card 9 leaves the runs overview's design out of battencli's package doc

**Location:** `_lyx/plan/09-batten-overview.md:42`

**Issue:** `PATTERN-documentation-lifecycle` requires a built design to live in its package's doc.
`docs/overview.md`'s "Documentation lifecycle" section says that when work lands, "the entry's design moves into the package's `doc.go`", and that a module's purpose and key design rationale then live in its Go package header comment.
battencli has no `doc.go`; its package comment is the header of `internal/battencli/cli.go`, and that comment describes each of the verbs the package adds beyond the shedverbs set: `lyx batten cleanup` and `lyx batten run <slug> --window`.
Card 9 builds D2 as a new battencli verb, but its only doc change is one bullet in `docs/overview.md`.
The run-selection, line, ordering and polling design, and the verb's bound, go into no package comment.
Card 7 does the matching step for the checklist (`loomcli/doc.go` states the checklist verb), and so do cards 8, 10 and 12 for their packages; card 9 is the one gap.
The decision record's docs constraint lists the packages it already expects to change, and does not override the pattern for battencli.

### [MEDIUM] FormatMinutes's hour and day forms are pinned by no test

**Location:** `_lyx/plan/05-statuspane-pane.md:47`

**Issue:** Card 5 defines `statuspane.FormatMinutes` with three output shapes (`12m`, `1h05m`, `2d03h`) and states it "has no test of its own", relying on `TestRenderChecklist` (card 7) and `TestRenderOverview` (card 9).
None of the rows either card names uses a duration of an hour or more.
Card 7's rows pin boxes, visit counts, halts and gotos; card 9's rows pin ordering, visibility and the unreadable and fit cases.
So the hour and day branches, and the minute-to-hour and hour-to-day boundaries, reach no planned test.
The card also leaves the behaviour it introduces uncovered until two cards later, against the plan template's rule that a card carries the coverage for the behaviour it introduces.
The no-test clause came in with round 1's fix for the duplicated format.
That fix assumed the renderer tests "already assert" the time strings, but no named row has a duration long enough to reach the hour or day forms, so the assumption does not hold.
On focus directive 2: the format is carried in one shared symbol, `statuspane.FormatMinutes`, created on card 5 and listed under `Uses:` by both cards 7 and 9, so there are no package-local copies.

### [MEDIUM] ValidateDisplay's size refusals and hidden exemption reach no named test

**Location:** `_lyx/plan/01-render-size-vocabulary.md:46`

**Issue:** The rubric's reachability rule asks every prescribed refusal or guard branch to name a test that reaches it.
Card 1 prescribes seven `ValidateDisplay` refusals and one exemption ("A `hidden` display is never refused for its size").
Card 1's `wantErr` rows reach only a top positional strand, a top sharing strand and `ShareDen` 0.
Card 2's `TestAddStrandLocked` and `TestUpdateStrandLocked` rows repeat the same three.
`own-window` keeps its existing coverage.
Four branches reach no test: the unknown `SizeKind`, `ShareNum < 0`, `ShareNum > ShareDen` and `MinRows < 1`.
The hidden exemption reaches none either.
No upstream check rules these inputs out.
Shuttle's `Validate` only defaults an empty anchor, and reed's `validateDisplay` passes the display straight to `render.ValidateDisplay`, so a spawner's bad `Size` arrives at these branches.

### [LOW] TestPartitionByAnchor row names an impossible display

**Location:** `_lyx/plan/01-render-size-vocabulary.md:67`

**Issue:** `render.Display.Anchor` is one `Anchor` value, so a strand is either `top` or `hidden`, never both.
The row "a hidden top content strand is dropped" cannot be built as written.
The behaviour the row evidently means, a hidden strand whose size declaration is ignored and which is dropped from placement, is what D3's "`hidden` with any kind: no pane" requires.

## Rejected

- Focus directive 1 (card 8's nested nil funcs): not a defect.
  `envkit.NilSeams` walks `shedrecipe.Env` recursively (`walkNil` descends into every struct field), and `InnerRunDeps` and `RowNoticeDeps` reach the env as `InnerRun`, `Land.Notices` and `Teardown.Notices`.
  So the only new nil func paths are exactly the three card 8 adds to `nilLegal`: `InnerRun.NoticeLog.Now`, `Land.Notices.NoticeLog.Now` and `Teardown.Notices.NoticeLog.Now`.
  `NoticeLog.Path` and `AttachCommand` are strings, which `walkNil` never reports.
  With those three in `nilLegal`, `TestFullEnv_FillsEverySeam`, battencli's `TestWire_EverySeamFilled` and loomcli's seam tests all skip them, so neither intentionally-nil list needs an entry.
- Focus directive 2 (shared time format): not a defect, as stated in the FormatMinutes finding: one symbol, `statuspane.FormatMinutes`.
- Focus directive 3 (card 9 helper tests): not a defect.
  `fitOverview` is covered by rows of `TestRenderOverview`, not by a test of its own.
  `TestOverviewSource_ReadsThePrimesRunsFromDisk` pins the disk-reading frame (child-status fallback, the recipe filter, the two-strike rule through the real cache), a behaviour the pure rendering test cannot reach.
  `TestOverviewCmd_RefusesOutsideThePrime` reaches the verb's refusal.
  The battencli budget arithmetic, 68 → 69 → 72, matches the four new top-level tests across cards 8 and 9.
- Focus directive 4 (card 1's untouched files against `render#placement`): not a defect.
  `placement` is declared in `layout.go`.
  Card 1's Intent says `layout.go` "changes only the comment of `placement.strip`", and listing `internal/reedengine/render#placement` under `Edit:` is that comment change, so the Intent and the target list agree.
- Card 2's title-row handling: `render` adding one row to a row-0 content cell, and `pinsForContent` subtracting one from the physically row-0 pin, together give a pin equal to the content rows. This matches D3 and is not double-counted.
- Round 1's four fixes hold: card 8 lists `nilLegal`, card 1's Intent words the `layout.go` comment change, `FormatMinutes` is one shared symbol, and `TestFitOverview` is gone. Round 1's fixer report disputed and deferred nothing.
- The test-budget arithmetic was checked against `isTopLevelTest` for loomcli (207 → 204 → 204), reedengine (255 → 256 → 257), statuspane, orchcli, termwindow, battenshed and loomrecipe; every row matches.
