# Card 2 — pattern-warp-root

**Edit:**
- `internal/pattern/pattern.go`
- `internal/pattern/doc.go`
- `internal/pattern/pattern_test.go`
- `internal/pattern/patternpath_test.go`
- `internal/pattern/leaf_enforcement_test.go`
- `contracts/stencils/pattern/pattern-directive-implementer.md`
- `contracts/stencils/pattern/pattern-directive-review-fix.md`
- `contracts/stencils/pattern/pattern-directive-orchestrator.md`
- `internal/websterengine/render.go`
- `internal/websterengine/recoverbatch.go`
- `internal/websterengine/runlevel.go`
- `internal/websterengine/template_test.go`
- `internal/loomengine/plan.go`
- `internal/loomengine/rework.go`
- `internal/loomengine/plan_test.go`
- `internal/loomengine/prompt_test.go`
- `internal/burlerengine/engine.go`
- `internal/burlerengine/engine_test.go`
- `internal/burlerengine/template_test.go`
- `cmd/lyx/pathrules_test.go`
- `internal/fabricengine/junction_pattern_integration_test.go`
- `internal/fabricengine/unwire_test.go`

**Intent:** Move PATTERN to the warp worktree root, inline the overview into every directive, and treat a whitespace-only overview as inactive (D3, D4, Operator additions 1–2).

`internal/pattern`:
- `File(worktreeRoot)` returns `<worktreeRoot>/PATTERN.md`.
  Add the background-directory counterpart as unexported constants: overview `PATTERN.md` and background directory `pattern`, each declared once.
  `PathspecFile` and `PathspecDir` are deleted, because card 1 removed their only consumer.
  The package no longer imports `internal/lyxdirs`.
  `leaf_enforcement_test.go`'s allowed set drops it, so the leaf stays at stdlib, `stencil` and `stencilstore`.
- `Directive(worktreeRoot, stencilsDir, role)` reads `File(worktreeRoot)`.
  An absent file, a directory in its place or whitespace-only content yields `("", nil)`.
  A read error on an existing file, or a stat error other than not-exist, is an error.
  Otherwise it reads the role's stencil, fills its `{{.pattern_overview}}` marker with the file's content verbatim (`stencil.Fill`; the marker is required in every directive stencil), and returns the result.
  The banner is stripped as today.
  `statFile` stays the test seam if the read path still needs one; otherwise switch it to an equivalent read seam.
- `doc.go`:
  - Rewrite "The active check is pure existence" for the new rule: the file at the warp worktree root is read, and whitespace-only is inactive.
  - Rewrite "Why the pointer stays relative": the overview is inlined because it is guaranteed seen, saves a read turn per agent and stays cache-stable, while background files stay relative `pattern/` pointers.
  - Drop the `PathspecFile and PathspecDir` section.

Directive stencils: replace the "Read _lyx/PATTERN.md in full" bullet with the `{{.pattern_overview}}` marker, placed where the agent reads it as the rule list.
Change the background bullet's literal to `pattern/` (for example "Read every background file under pattern/ that the entries above link to and that touches …").
The pointer stays a fixed literal, never interpolated.
Each banner drops "must stay marker-free" and states the one marker and who fills it.

Call sites pass the warp worktree root, never the anchor:
- `loomengine` passes `layout.WorktreePath()`.
- `burlerengine` passes `e.geom.WorktreeRoot`.
- `websterengine.RenderMasterPrompt` passes its `worktreeRoot`.
- `RenderRecoveryPrompt` replaces its `anchorRoot` parameter with the worktree root the recovery agent edits; `recoverbatch.go` and `runlevel.go` follow.
  Drop the now-unused `anchorRoot` parameter where it fed only the directive.

Tests:
- The fixtures that plant `_lyx/PATTERN.md` plant `PATTERN.md` at the fixture's worktree root instead.
- `pattern_test.go` covers:
  - the overview content appearing verbatim in every role's directive;
  - a whitespace-only file yielding `""`;
  - an absent file yielding `""`;
  - a directory in place of the file yielding `""`;
  - an unreadable file being an error;
  - the background pointer staying the literal `pattern/` in every role's text.
- The edge-case tests that pinned an empty file as active are rewritten to the new rule.
- `TestPlanSpec_PatternDirectiveAnchoredUnderAnchorPath` becomes a subpath-anchored case proving the worktree root, not the anchor, is read.
- `pathrules_test.go` drops the `pattern.File` row, since the overview is no longer `_lyx` content; its comment follows.
- `junction_pattern_integration_test.go` tests fabric's pollution handling of a tracked file under warp's `_lyx`, so it keeps its fixture file but spells its expected path locally (for example `lyxdirs.LyxDirName + "/PATTERN.md"`) instead of reading `pattern.PathspecFile`.
- `unwire_test.go`'s comment that names `_lyx/PATTERN.md` as PATTERN's home is corrected.

**ImpactSummary:** Changes Directive's path root and output across its four engine call sites plus webster's two render callers; no CLI surface changes.
