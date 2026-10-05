# Skills and stencils: roles get procedure from stencils and skills from lyx

All 16 batches reported `done` on the first fork, with no recovery.

- **01–02, skill loading:** `shuttleengine.Spec` gained `Skills` and `SkillLoadTimeout`, `Launch` gained `PromptLine`, and the optional `SkillLoader` capability was added.
  The Claude engine implements it by typing each skill as a slash command, detecting an unknown skill from the pane, and deferring the prompt pointer until the skills are loaded.
- **03, parent directive:** the new `internal/parentdirective` package renders the parent, operator-ban and no-parent variants from three stencils under `contracts/stencils/parent/`.
- **04–09, roles:** the loom producers (Discussion, Plan, Rework), the Webster master and recovery, burler rounds, the Bouncer judge and seed, the treadle roles, and the conflict, describe and friction roles each declare a skill list beside their role constant.
  Each takes a told parent name and renders the directive in its stencil.
- **10–11, driver:** the loom driver launches from `shed-template-driver.md`, and the `ly-drive` skill was removed and its mentions reworded.
- **12–15, orch:**
  - The orch gained a role stencil, a note template and one-line pointers.
  - `cycle` and `distill` both write the note first.
  - Shuttle reads `CompactedSince`.
  - The watcher reloads skills, role and note after a clear, a compaction and an auto-compaction.
- **16, PATTERN:** `PATTERN-role-skills-typed` and `PATTERN-parent-directive` were added, with stencil scans in `contracts/stencils`.

Reported deviations from the declared file-ops, all informational:

- **05:** `internal/testkit/shuttlefake/shuttlefake.go` and a new `internal/hubgeom/webstergeom_integration_test.go`.
- **06:** `internal/burlerengine/template_test.go`.
- **07:** `internal/shedadapters/bouncerprompt_test.go`.
- **08:** `internal/treadleengine/seam_enforcement_test.go` and `template_test.go`.
- **09:** the two `landingDeps` call sites in `internal/loomcli/arm.go` and `sharedbootstrap.go`, their integration and verify tests, `internal/landingshed/finalize_integration_test.go`, and the seam allowlists of `mergeresolve`, `landingshed` and `frictionengine`.
- **10:** `contracts/stencils/citation_enforcement_test.go` and `internal/loomcli/integration_driverbootstrap_test.go`.
- **11:** `internal/loomcli/start.go`, `driverprompt.go` and `bootstrap_test.go`.
- **12:** `internal/orchengine/state_test.go`.
- **15:** `internal/orchcli/start.go` and `internal/orchengine/watchloop_test.go`.
