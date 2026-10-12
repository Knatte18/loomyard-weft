# Card 1 — render accepts the own-window display and its group fields

**Edit:**
- `internal/reedengine/render#Display`
- `internal/reedengine/render#Anchor`
- `internal/reedengine/render#AnchorOwnWindow`
- `internal/reedengine/render#ValidateDisplay`
- `internal/reedengine/render#partitionByAnchor`
- `internal/reedengine/render#TestFixedHeightPinsMatchesRulesPlacedHeights`

**Intent:**
D1 and D2's add-time refusal, at the render layer: `own-window` stops being a reserved, refused anchor and becomes a declared placement that window 0's layout never places.

`internal/reedengine/render/types.go`:

- `Display` gains two fields, both persisted verbatim in the reed.json strand record, so both carry `omitempty` (`PATTERN-tolerant-records`):
  - `Window string \`json:"window,omitempty"\``: the name of the window group the spawner puts the strand in; empty for a strand that wants a window to itself.
  - `WindowSize int \`json:"windowSize,omitempty"\``: the number of strands the spawner puts in that group; 0 reads as 1.
  Their doc comments say they are read only for an `own-window` strand, and that reed decides from them whether the group shares one window (card 5).
- `AnchorOwnWindow`'s doc comment says it gives the strand a tmux window of its own, or its group's shared window, outside window 0, and that window 0's layout never places it.
- `Anchor`'s doc comment says render recognizes exactly the four values, and drops "own-window is reserved and refused".
- The package comment's vocabulary sentence becomes "The vocabulary is top, below-parent, own-window and hidden; own-window and hidden strands are never placed in the window this package lays out."

`ValidateDisplay`:

- The `AnchorOwnWindow` case returns nil like `AnchorHidden` does, since the size declaration is ignored for a strand that is never placed, except that a `WindowSize` below 0 is refused with an error naming the field: `anchor "own-window": windowSize %d must not be negative`.
- The doc comment drops the reserved own-window refusal and names the negative `WindowSize` one.

`partitionByAnchor`'s default-case comment drops "(deferred)": an `own-window` strand lives outside this window and is never placed by this policy.

`planCells` calls `ValidateDisplay` for every strand, so a persisted own-window strand no longer fails `Rules` or `FixedHeightPins`.
`internal/reedengine`'s own `validateDisplay` still refuses `own-window` at the op boundary until card 4 opens it, so no strand can be added with it yet.

`TestFixedHeightPinsMatchesRulesPlacedHeights`:

- The row `AnchorOwnWindowYieldsNilNotAPanicMatchingRulesError` becomes `AnchorOwnWindowIsNeverPlacedNorPinned`: a live own-window strand with a pane id beside a below-parent strand yields the below-parent strand's pins only, and `wantErr` false.
- A new row `NegativeWindowSizeIsRefusedLikeRules`: an own-window strand with `WindowSize: -1` makes both `FixedHeightPins` and `Rules` return an error, exactly as the row loop already checks for `wantErr` rows.

`TestPartitionByAnchor`'s existing `OwnWindowNotPlaced` row already pins that an own-window strand is not placed and needs no change.

**ImpactSummary:** Render-only: `ValidateDisplay` has two callers, `planCells` and reedengine's `validateDisplay`, which keeps refusing own-window until card 4; one pins row changes meaning and one is added.

**Commit:** 1: Render accepts the own-window display with its window group fields
