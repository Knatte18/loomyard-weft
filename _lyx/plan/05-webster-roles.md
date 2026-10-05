# Card 5 — Webster master and recovery load their skills and render the parent directive

**Edit:**
- `internal/websterengine/geometry.go`
- `internal/websterengine/roles.go`
- `internal/websterengine/render.go`
- `internal/websterengine/runlevel.go`
- `internal/websterengine/recoverbatch.go`
- `internal/websterengine/doc.go`
- `internal/hubgeom/webstergeom.go`
- `contracts/stencils/webster/webster-template-master.md`
- `contracts/stencils/webster/webster-prefix-recovery.md`
- `internal/websterengine/render_test.go`
- `internal/websterengine/roles_test.go`
- `internal/hubgeom/webstergeom_test.go`

**Uses:**
- `internal/parentdirective#Directive`
- `internal/parentdirective#MarkerName`
- `internal/hubgeom#ResolveParent`
- `internal/websterengine#Geometry`

**Intent:**
Move the Webster master and the cold-start recovery session onto D1 and D9.

`websterengine.Geometry` gains `ParentName`, told like every other geometry field.
`hubgeom.WebsterGeometry` fills it from `ResolveParent`, logging a warning and leaving it empty when the parent is unresolvable, as `reedGeometry` does; `standalonegeom` leaves it empty, so a standalone run renders the no-parent variant.
The geometry reaches both the master (through `RunDeps.Geom`) and the recovery spawn in the separate `lyx webster` process (through `RecoverDeps.Geom`), so no other wiring changes.

Skills (D2), declared in `roles.go` beside `RoleMaster` and `RoleRecovery`: `scribe:prose`, `scribe:code-quality`, `scribe:testing` for both.
The master spec in `runlevel.go` and the recovery spec in `recoverbatch.go` set them.
Forks get no skills and no directive: they inherit the master's context.

`RenderMasterPrompt` and `RenderRecoveryPrompt` fill `parentdirective.MarkerName` with `parentdirective.Directive(stencilsDir, geom.ParentName, false)`, taking the parent name as one more told argument.
`webster-template-master.md` and `webster-prefix-recovery.md` carry `{{.parent_directive}}` near their top; `webster-prefix-fork.md` and the `webster-body-*` files carry none.
`RenderForkPrompt` and `RenderVerifyFixPrompt` are unchanged.

Tests: both prompts carry the directive with the parent's name and the no-parent variant with an empty one; the fork prompt carries none; the role skill lists are the D2 lists; `WebsterGeometry` sets `ParentName` from a fixture's origin record, the way `hubgeom_test.go` already tests `reedGeometry`'s parent.
`doc.go` names the skills and the directive in its prompt section.

**ImpactSummary:** webster's two opening prompts, its two spawn specs and one geometry field; every `RenderMasterPrompt`/`RenderRecoveryPrompt` caller is inside websterengine.

**Commit:** 5: webster: master and recovery load skills and render the parent directive
