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
- `internal/burlerengine/geometry.go`
- `internal/websterengine/geometry.go`
- `internal/hubgeom/hubgeom.go`
- `internal/hubgeom/hubgeom_test.go`
- `internal/hubgeom/webstergeom.go`
- `internal/hubgeom/webstergeom_test.go`
- `internal/standalonegeom/burlergeom.go`
- `internal/standalonegeom/webstergeom.go`
- `internal/standalonegeom/standalonegeom_test.go`
- `cmd/lyx/pathrules_test.go`
- `internal/fabricengine/junction_pattern_integration_test.go`
- `internal/fabricengine/unwire_test.go`
- `internal/fabricengine/unwire.go`
- `internal/fabricengine/junction.go`

**Intent:** Move PATTERN to the warp worktree root, inline the overview into every directive, and treat a whitespace-only overview as inactive (D3, D4, Operator additions 1–2).
Bound on the inactive rule: only whitespace-only content is inactive; any other content, however malformed, is inlined verbatim, so format violations stay the job of card 6's checker.
An emptied loomyard overview cannot slip through, because card 6's real-file test fails on an empty overview and on one with no entry bullet.

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

Call sites pass the warp worktree root, never the anchor.
In hub mode the burler and webster geometries' `WorktreeRoot` is the anchor path (`hubgeom.BurlerGeometry` and `hubgeom.WebsterGeometry` set it to `l.AnchorPath()`, and their comments forbid converging it), so it is wrong for a subpath-anchored hub and neither engine may read `PATTERN.md` through it.
Each of the two engines' `Geometry` gains a told `RepoRoot` field, the directory holding `PATTERN.md` and `go.mod`:
- `hubgeom`'s burler and webster geometry constructors fill it with `l.WorktreePath()`.
- `standalonegeom`'s burler and webster geometry constructors fill it with the target, as for their `WorktreeRoot`.
- A `Geometry` literal that leaves it empty gets no directive, through `Directive`'s empty-path case.
- The existing `WorktreeRoot` semantics do not change, and the `hubgeom` comments that explain why they differ stay.

Call sites:
- `loomengine` passes `layout.WorktreePath()`.
- `burlerengine` passes `e.geom.RepoRoot`.
- `websterengine.RenderMasterPrompt` and `RenderRecoveryPrompt` take the repo root as the argument they pass to `Directive`.
  `RenderMasterPrompt` replaces its now-unused `anchorRoot` parameter with `repoRoot`, and `RenderRecoveryPrompt` replaces its `anchorRoot` parameter with `repoRoot` and keeps `promptWorktreeRoot`, which feeds the card pointers.
  `runlevel.go` and `recoverbatch.go` pass `deps.Geom.RepoRoot`, and `template_test.go`'s calls follow.

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
- `unwire_test.go`'s comment, `unwire.go`'s `WeftContent` comment and `junction.go`'s comment on the real-directory refusal all name `_lyx/PATTERN.md` as PATTERN's home; each is corrected to say PATTERN no longer lives under `_lyx`.
- `hubgeom_test.go`, `webstergeom_test.go` and `standalonegeom_test.go` assert the new `RepoRoot` field: the worktree path on a subpath-anchored hub location, where it differs from the anchor path, and the target in standalone mode.
- The burler and webster directive tests gain a subpath-anchored case, with `PATTERN.md` planted at the repo root and not at the anchor, proving the directive renders.

**ImpactSummary:** Changes Directive's path root and output across the loom, burler and webster call sites, and adds one told field to two engine geometries, filled by the hub and standalone constructors; no CLI surface changes.
