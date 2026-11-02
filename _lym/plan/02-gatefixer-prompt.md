# Card 2 — gatefixer-prompt

**Create:**
- `contracts/stencils/gatefix/gatefix-frame-cold.md`
- `contracts/stencils#GatefixFrameCold`
- `internal/gatefixer#FrameStencil`
- `internal/gatefixer#PromptInputs`
- `internal/gatefixer#RenderPrompt`
- `internal/gatefixer#TestRenderPrompt`

**Extends:**
- `contracts/stencils#entries`
- `contracts/stencils#roleOpeningStencils`
- `contracts/stencils#TestStencils_TopLevelMarkers`
- `internal/gatefixer/doc.go`

**Uses:**
- `internal/testkit/stencilkit#Seed`
- `internal/websterengine#composeVerifyFixTemplate`
- `contracts/stencils#TestStencils_OpeningStencilsRenderParentDirective`
- `contracts/stencils#TestStencils_OpeningStencilsRenderEditDirective`

**Intent:**
Add the shared cold frame every fixer's prompt opens with, and the renderer that joins it to one caller body stencil (D1, D5).

`contracts/stencils/gatefix/gatefix-frame-cold.md` is `webster-prefix-verify-fix-cold.md` made caller-neutral: a strand starting cold and inheriting nothing; its only job is to fix the failures the report at `{{.report_path}}` lists, as far as its body's edit scope allows, and write its reply; the orientation steps read the report in full and the source behind each failure before any edit; the final action writes `{{.reply_path}}` with Write, naming each failure fixed and each left with why, and ends the turn right after it.
It names no caller, no `lym webster` verb and no plan file, and holds `{{.parent_directive}}`, `{{.edit_directive}}`, `{{.pattern_directive}}` and `{{.friction_directive}}` where the webster prefix holds them, with no `{{if}}` or `{{range}}`.
Its leading comment names the markers and that `internal/gatefixer`'s `RenderPrompt` joins it ahead of one caller body.
It is embedded as `GatefixFrameCold` beside the other stencils and registered in `entries` under `gatefix-frame-cold`, its family directory `gatefix` read from the name as `stencilstore.RelPath` reads every family.
`roleOpeningStencils` gains `"gate-fixer": {"gatefix-frame-cold"}`, so the parent and edit directive tests hold the frame to both markers; `TestStencils_TopLevelMarkers` gains its row: `edit_directive`, `friction_directive`, `parent_directive`, `pattern_directive`, `reply_path`, `report_path`.

`PromptInputs` carries `StencilsDir`, `BodyStencil` (the caller body's stencil name), `BodyValues map[string]string`, `ReportPath`, `ReplyPath`, `RepoRoot`, `PatternChanges []string`, `ParentName` and `NotePath`.
`RenderPrompt(in)` reads `FrameStencil` and `in.BodyStencil` through `stencilstore` at call time, joins them frame first as `composeVerifyFixTemplate` joins prefix and body, and fills `report_path`, `reply_path` and every body value, the pattern directive from `pattern.Directive(in.RepoRoot, in.StencilsDir, pattern.RoleImplementer, in.PatternChanges)`, the parent directive from `parentdirective.Directive(in.StencilsDir, in.ParentName, false)` (its no-parent variant when `ParentName` is empty), the edit directive from `editdirective.Directive`, and the friction directive from `friction.Directive(in.NotePath, ...)` when `NotePath` is set, with `stencil.FillOptional` taking the pattern, friction and parent markers as optional, as `RenderVerifyFixColdPrompt` does.
It refuses an empty `ReplyPath` or `ReportPath`, naming the field; that is a fail-loud wiring guard.
A body value naming `report_path` or `reply_path` is overwritten by the frame's own.

`TestRenderPrompt`, tier0, renders over a stencils dir from `stencilkit.Seed`, which holds the frame and the directive stencils `RenderPrompt` fills, with a test body of one marker written into it: the report and reply paths and the body value appear; a body value named `report_path` is overwritten by the frame's own; the parent directive's no-parent variant renders with an empty `ParentName` and the named variant otherwise; the friction directive renders with `NotePath` set and is absent without it; an empty `ReplyPath` and an empty `ReportPath` are each refused naming the field; a missing body stencil names it.

**ImpactSummary:** one stencil registered and one role mapped in the stencil tables, read only by the new renderer; no existing prompt changes.
