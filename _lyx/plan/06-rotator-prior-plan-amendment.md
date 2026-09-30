# Card 6 — rotator-prior-plan-amendment

**Edit:**
- `internal/loomshed#NewPlanDirRotator`
- `internal/loomshed#rotateStalePlanDir`
- `internal/loomshed/planwrite_test.go`
- `internal/loomshed/doc.go`
- `internal/shedrecipe#planWriteEntry`
- `internal/shedrecipe/entries_planwrite_test.go`
- `internal/loomrecipe/fixture_test.go`
- `internal/loomengine/plan.go`

**Uses:**
- `plan:internal/loomengine#PriorPlanBlock`
- `plan:contracts/stencils#LoomTemplatePriorPlan`

**Intent:**
Make Plan-Write's fresh-spawn rotation tell the new session where the prior plan went (decision record D1, D3).
The rotation itself stays: every top-level `.md` file still moves into a fresh `archive-<stamp>[-N]` directory, and the agent always runs.

`rotateStalePlanDir` returns the archive directory it created and the names of the files it moved, alongside its error; it returns an empty directory and no names when nothing moved (absent plan directory, or no top-level `.md` file), creating nothing.

`NewPlanDirRotator` gains a `stencilsDir string` parameter, told by its caller.
Its closure returns the prompt amendment: empty when nothing moved, otherwise a blank line followed by `loomengine.PriorPlanBlock(stencilsDir, archiveDir, moved)`.
A render failure is returned as the closure's error, so the producer aborts before spawning a session that would silently rewrite the plan.
The plan directory stays resolved through `planparser.PlanDir(anchorPath)`, and no `_lyx`/`.lyx` literal is added.
Update both functions' doc comments.

`planWriteEntry` validates `env.StencilsDir` with `requireAbsRoot("PlanWrite", "StencilsDir", ...)` and passes it to `NewPlanDirRotator`; update its doc comment's list of Env fields it reads.

Docs:

- `internal/loomshed/doc.go`: state that the Plan-Write rotation appends a prior-plan block naming the archive to the respawned session's prompt.
- `internal/loomengine/plan.go`'s file comment: replace the sentence calling stale-plan rotation "the future loom phase machine's responsibility" with one naming `loomshed.NewPlanDirRotator`, run as the Plan-Write producer's fresh-spawn preparation, as its owner; verifying the decision record exists stays outside `PlanSpec`.

Tests in `planwrite_test.go`, seeding `stencils.LoomTemplatePriorPlan` into a temp stencils directory:

- Prior files present: every top-level `.md` file moves into `archive-<stamp>`, and the amendment contains that directory's absolute path and every moved file name.
- Absent plan directory, and a plan directory with no top-level `.md` file: the amendment is empty and no archive directory is created.
- Same-second collision (the `archive-<stamp>` directory already exists): the amendment names the `-1`-suffixed directory the files moved to, and not the colliding one.
- A stencils directory missing the stencil, with prior files present: the closure returns an error.

Tests in `entries_planwrite_test.go`:

- An empty `env.StencilsDir` fails construction, the error naming `PlanWrite` and `StencilsDir`.
- A stale `.md` file in the plan directory under `env.AnchorPath`, with a minimal `loom-template-prior-plan` stencil written into `env.StencilsDir`: one `Call` hands the shuttle a spec whose prompt is the composed prompt followed by the rendered block.

Fixture in `internal/loomrecipe/fixture_test.go`:
`buildSequenceFixture` seeds a plan with `seedPlanFixture` before Plan-Write runs, so the rotation moves files and the rotator now renders the prior-plan block from `env.StencilsDir`.
`stencilstore.Read` never falls back to the embedded default, so add `"loom-template-prior-plan": stencils.LoomTemplatePriorPlan` to `seedBouncerStencils`' seed map and update its doc comment's list of seeded stencils.
Without it, every sequence test built on that fixture (`TestSequence_FullRunBlocksAtPublish`, the `resume_test.go` and `frictionreflect_test.go` runs) fails at Plan-Write with the render error.

**ImpactSummary:** `NewPlanDirRotator` gains a parameter and its single production caller, `planWriteEntry`, now requires `Env.StencilsDir`, which every loom wiring already sets; a rotation that moves files now also requires `loom-template-prior-plan` in that directory, and the one test fixture that seeds a plan before Plan-Write (`loomrecipe`'s `buildSequenceFixture`) is updated to seed it; rotation behaviour on disk is unchanged.
