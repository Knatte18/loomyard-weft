# Card 2 — reed-content-strands

**Create:**
- `internal/reedengine#validateDisplay`
- `internal/reedengine#paneSplit`

**Delete:**
- `internal/reedengine#validateAnchor`

**Edit:**
- `internal/reedengine#Strand`
- `internal/reedengine#toRenderStrands`
- `internal/reedengine#Engine.toRenderInputs` -> `func (e *Engine) toRenderInputs(st *ReedState, live []LivePane, titleRow bool) renderInputs`
- `internal/reedengine#Engine.planLayout` -> `func (e *Engine) planLayout(st *ReedState, live []LivePane, box render.Box, titleRow bool) (layout, focus string, err error)`
- `internal/reedengine#Engine.fixedHeightPins` -> `func (e *Engine) fixedHeightPins(st *ReedState, live []LivePane, box render.Box, titleRow bool) []render.Pin`
- `internal/reedengine#Engine.contentPinsLocked` -> `func (e *Engine) contentPinsLocked(st *ReedState, live []LivePane, box render.Box, titleRow bool) []render.Pin`
- `internal/reedengine#Engine.applyLayoutLockedOpts`
- `internal/reedengine#Engine.AttachArgv`
- `internal/reedengine#Engine.addStrandLocked`
- `internal/reedengine#Engine.updateStrandLocked`
- `internal/reedengine#planPaneTarget` -> `func planPaneTarget(st *ReedState, live []LivePane, display render.Display) (paneSplit, error)`
- `internal/reedengine#Engine.launchStrandLocked`
- `internal/reedengine#Engine.splitPaneBelowLocked`
- `internal/reedengine/template_posix.yaml`
- `internal/reedengine/template_windows.yaml`
- `docs/overview.md`
- `internal/reedengine#liveRenderStrands`
- `internal/reedengine#TestPlanLayout_MatchesRenderRules`
- `internal/reedengine#TestPlanLayout_StaleSelvagePaneIDNeverEmittedAsLayoutCell`
- `internal/reedengine#TestAttachArgv_ChainedArgv`
- `internal/reedengine#wantChainedAttachArgv`
- `internal/reedengine#expectedLayoutForCurrentBox`
- `internal/reedengine#TestPlanPaneTarget`
- `internal/reedengine#TestAddStrandLocked`
- `internal/reedengine#TestUpdateStrandLocked`
- `internal/reedengine/doc.go`

**Intent:**
Carry card 1's size declaration through reed: persist it, plan with it, refuse a bad one at the op boundary, and put a top-anchored strand's pane physically at the top, since `render.Rules` resequences cells by physical pane order (`resequenceByPaneOrder`) and a top cell can only land on top if its pane does.

Persistence: `Strand` gains `ContentRows int` tagged `json:"contentRows,omitempty"`; `Display` already persists verbatim, `Size` with it.
A strand persisted by an older lyx has neither and lays out positionally.
`toRenderStrands` copies `ContentRows` onto `render.Strand`.
Card 3 is the only writer of `ContentRows`.

Title row: `toRenderInputs` sets `render.Params.TitleRow` from its new `titleRow` argument, and `planLayout` and `fixedHeightPins` pass theirs through, so the layout and its pins are planned from the same title-row answer.
`applyLayoutLockedOpts` and `AttachArgv` each call `readBorderTitleRowLocked` once, after their guards and before planning, and hand the answer to both `planLayout` and `contentPinsLocked`.
`contentPinsLocked` no longer reads it itself (its `windowTarget` parameter goes), and still adjusts the row-0 pin through `pinsForContent`.
The test callers of `planLayout` pass `false`, reproducing their expectations unchanged.

Validation: `validateDisplay` replaces `validateAnchor` at both op boundaries (`addStrandLocked`, `updateStrandLocked`): the anchor must be `below-parent`, `hidden` or `top` (an empty or mistyped one is refused as today, `own-window` with today's deferred message), and `render.ValidateDisplay` must accept the whole display, so a top strand that is not content-sized, or a content size with a zero denominator, is refused before any pane is launched or record registered.

Top placement: `planPaneTarget` takes the new strand's display and returns a `paneSplit` (`target string`, `above bool` for `-b`, `full bool` for `-f`).
A strand that is not top-anchored gets today's three tiers unchanged, `above` set exactly where `insertAbove` was.
A top-anchored strand splits below the physically lowest alive pane bound to another top-anchored strand of `st`, so top strands stack in insertion order; with none, it splits the window's physically top-most present pane with `-f -b`, a full-width split above every pane that takes its rows from the whole window, so a collapsed one-row top pane cannot make it fail, and Selvage stays bottom-most.
`launchStrandLocked` passes `s.Display` and builds `split-window` from the three fields; its `-b` comment covers `-f`.
`splitPaneBelowLocked`'s doc comment no longer says Selvage is the only split that must land at a physical edge, and that strands have no positional requirement: a top strand's split must land on top too.

Docs: `doc.go`'s layout-rule paragraph is rewritten to D3 (positional, content and sharing kinds, the top anchor, the free height, the expanding cells, the clamp order and the fixed-height panes the hook pins), keeping the sentence that a window whose strands declare nothing lays out as before.
The `collapsed_rows` comment in both reed templates says it sizes the positional strands, all of which collapse while a sharing strand is live, and `docs/overview.md`'s reed bullet (the "height is derived by one rule, not caller-set" sentence) names the size declaration a caller may hand reed beside the anchor and focus.

Tests:
- `TestPlanLayout_MatchesRenderRules` gains a `titleRow` field and a row with a top content strand (`ContentRows` 4) above a positional strand, laid out with `titleRow` true, whose expected layout is `render.Rules` over `liveRenderStrands` with `Params.TitleRow` true; `liveRenderStrands` copies `ContentRows`, so a `toRenderStrands` that drops the count or a `planLayout` that drops the title row yields a different body.
- `TestPlanPaneTarget` gains `display`, `strands` and `wantFull` fields, the existing rows keeping the zero display, and three rows: a top strand with no top strand bound splits the top-most pane (`%1` at Top 0 in a `%1`/`%2`/Selvage window) above and full; a top strand with a live top strand bound to `%1` splits `%1` below, not full; a top strand with only Selvage present splits Selvage above and full.
- `TestAddStrandLocked`'s rejected rows gain a top positional display, a top sharing display and a content display with `ShareDen` 0; `TestUpdateStrandLocked` gains a top positional row, its display left unchanged.

**ImpactSummary:** reedengine-internal: `planLayout`, `fixedHeightPins`, `contentPinsLocked`, `toRenderInputs` and `planPaneTarget` re-signed with every caller and test caller on this card; no exported signature changes.
