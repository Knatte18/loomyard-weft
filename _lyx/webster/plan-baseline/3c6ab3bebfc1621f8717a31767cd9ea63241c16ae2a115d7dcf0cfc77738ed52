# Card 1 — render-size-vocabulary

**Create:**
- `internal/reedengine/render#AnchorTop`
- `internal/reedengine/render#SizeKind`
- `internal/reedengine/render#SizePositional`
- `internal/reedengine/render#SizeContent`
- `internal/reedengine/render#SizeSharing`
- `internal/reedengine/render#Size`
- `internal/reedengine/render#ValidateDisplay`
- `internal/reedengine/render#cellRole`
- `internal/reedengine/render#roleCollapsed`
- `internal/reedengine/render#roleContent`
- `internal/reedengine/render#roleExpanding`

**Edit:**
- `internal/reedengine/render#Anchor`
- `internal/reedengine/render#Display`
- `internal/reedengine/render#Strand`
- `internal/reedengine/render#Params`
- `internal/reedengine/render#placement`
- `internal/reedengine/render#partitionByAnchor`
- `internal/reedengine/render#stackHeights`
- `internal/reedengine/render#clampToFit` -> `func clampToFit(heights []int, roles []cellRole) []int`
- `internal/reedengine/render#cellPlan`
- `internal/reedengine/render#planCells`
- `internal/reedengine/render#Rules`
- `internal/reedengine/render#FixedHeightPins`
- `internal/reedengine/render#TestRulesGolden`
- `internal/reedengine/render#TestFixedHeightPinsMatchesRulesPlacedHeights`
- `internal/reedengine/render#TestPartitionByAnchor`
- `internal/reedengine/render#TestDisplayDecodesRetiredKeysIgnored`
- `internal/reedengine/render#TestDisplayMarshalsWithoutRetiredKeys`

**Intent:**
Give render the D3 size vocabulary and height rules, in its policy layer only (`PATTERN-leaf-packages`: render stays a pure leaf, and no mechanics in `layout.go` or `checksum.go` change; `layout.go` changes only the comment of `placement.strip`, which now names every collapsed positional cell and every content cell).

Vocabulary, in `types.go`:
- `AnchorTop` joins the closed anchor set; the `Anchor` comment and the package comment name four values, `own-window` still reserved.
- `Size` is a strand's size declaration: `Kind SizeKind` (`json:"kind,omitempty"`), the preferred share of the free height as `ShareNum`/`ShareDen int` (`json:"shareNum,omitempty"`, `json:"shareDen,omitempty"`), and `MinRows int` (`json:"minRows,omitempty"`), the last three read for `SizeContent` only.
  The share is an integer fraction so `floor(share × free height)` is exact: one third is 1/3, never 0.33.
- `Display` gains `Size Size` tagged `json:"size,omitzero"`, so a strand that declares nothing marshals exactly as before and a record written by an older lyx decodes as `SizePositional`.
- `Strand` gains `ContentRows int`, the strand's reported visible content line count (0 means not reported yet, read as the strand's `MinRows`).
- `Params` gains `TitleRow bool`: the window draws a pane-border title row at window row 0.

`ValidateDisplay` (policy layer) refuses, returning an error naming the display's anchor and kind: `own-window`, `top` with a kind other than `SizeContent`, an unknown `SizeKind`, and a content size with `ShareDen < 1`, `ShareNum < 0`, `ShareNum > ShareDen` or `MinRows < 1`.
A `hidden` display is never refused for its size: its declaration is ignored, as D3 states.
`planCells` calls it for every strand in place of today's own-window loop and wraps its error with the strand's GUID in the form `render: strand <guid>: <error>`, so `Rules` and `FixedHeightPins` refuse exactly what it refuses and still name the offending strand (the `FixedHeightPins` nil-on-error contract is unchanged).

Placement: `partitionByAnchor` keeps its signature and returns the placed cells top-anchored strands first, in insertion order, then the below-parent stack in insertion order; hidden, not-live and paneless strands are dropped as today.
Its doc comment and the file comment of `policy.go` say so.

Heights, in `stackHeights` over that ordered list, with `F` the stack box height (the window's box less the Selvage band and its divider) and `usable = F - (n-1)` as today:
- A content cell's height is `min(R, max(m, floor(F × ShareNum / ShareDen)))`, with `m` its `MinRows` and `R` its `ContentRows`, or `m` while that is 0; the first cell of the list adds one row when `p.TitleRow`, so a pane of N content rows at window row 0 shows N lines.
- Expanding cells: every live sharing cell when there is one; else the bottom-most positional cell when there is one; else the bottom-most cell of the list, whatever its kind (a content cell taking it exceeds its maximum).
- Every positional cell that is not expanding takes `p.CollapsedRows` (floored at 1, as today); so while a sharing cell is live every positional cell collapses.
- The expanding cells split `usable` minus every fixed cell evenly, each taking the floor of the share and the bottom-most expanding cell the remainder rows.
- `clampToFit` takes a `cellRole` per cell instead of `isCollapsed`/`bottomIdx` and reclaims rows for a too-short window from collapsed cells first, then content cells down to one row each, then expanding cells from the bottom up, every height kept positive.
- `placement.strip` is true for every collapsed positional cell and every content cell, the expanding content cell included, so `FixedHeightPins` reports them all as D3's fixed-height panes; its comment and `FixedHeightPins`' and `Rules`' doc comments say which cells pin.
- `height.go`'s file comment states these rules in place of the one positional rule, and `cellPlan.ordered`'s comment says it holds the placed cells, top-anchored first.

A window of positional strands only takes exactly today's heights, so its layout string is byte-identical: every existing `TestRulesGolden`, `TestRulesPaneOrder` and `TestStackHeights` row passes unchanged, which is the pin for that constraint.

Tests, as rows of the existing tests (no new top-level test, so render's budget row stays):
- `TestRulesGolden` gains a `titleRow bool` field fed into `Params.TitleRow` and rows that each pin one D3 outcome by its exact layout body: a top content strand (share 1/3, min 3, `ContentRows` 2) above a positional strand with the title row, which takes 3 rows over a box of 30; the same without a reported count, taking its minimum plus the title row; a below-parent content strand between two positional ones; one positional and two live sharing strands, the positional collapsed and the two equal; a third sharing strand, all three equal; a sharing split whose remainder row goes to the bottom-most sharing strand; two sharing strands that are not live, where the bottom-most positional strand expands again; a window whose only strand is a content strand, with a Selvage band, taking every row of the stack; a box too short for every minimum, where the collapsed cell yields before the content cell and every cell stays positive; and a box shorter still, where the collapsed and content cells are both down to one row and the expanding cell gives up the rest, pinning the third stage of the clamp order.
- `TestFixedHeightPinsMatchesRulesPlacedHeights` gains rows: content cells pinned at their placed heights beside the Selvage pin, a sharing window pinning the collapsed positional cell and no sharing cell, and `wantErr` rows for a top-anchored positional strand, a top-anchored sharing strand, and below-parent strands carrying a content size with `ShareDen` 0, an unknown `Size.Kind` such as `fixed`, a content size with `ShareNum` -1 / `ShareDen` 3 / `MinRows` 3, one with `ShareNum` 2 / `ShareDen` 1 / `MinRows` 3, and one with `ShareNum` 1 / `ShareDen` 3 / `MinRows` 0, each where `Rules` errors and the pins are nil; and a non-error row where a hidden strand carrying a content size with `ShareDen` 0 lays out and pins exactly as the same window without that strand.
- `TestPartitionByAnchor` gains a row where a top strand inserted after a below-parent strand is placed first, and a hidden strand carrying a content size is dropped.
- `TestDisplayDecodesRetiredKeysIgnored` also asserts that a record with no `size` key decodes with a zero `Size`; `TestDisplayMarshalsWithoutRetiredKeys` also asserts that a zero `Size` marshals no `size` key, the on-disk contract `types_test.go` pins, and that a content `Size` round-trips through JSON unchanged.

**ImpactSummary:** render-internal: `Rules` and `FixedHeightPins` keep their signatures; reedengine reaches the new fields only from card 2 on; positional-only layouts are unchanged.
