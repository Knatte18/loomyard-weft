# Card 14 — driver-docs

**Prosa:**
- `docs/overview.md`
- `internal/sheddriver/doc.go`
- `internal/loomcli/doc.go`
- `internal/battenshed/doc.go`

**Intent:**
Bring the module docs in line with the moved driver, the `shed` config module and batten's driver body (CLAUDE.md "Docs land in the same commit"; `PATTERN-documentation-lifecycle`: a built design lives in its package's `doc.go`).
No code changes.

- `docs/overview.md`:
  - the module tree gains `internal/sheddriver/` beside `internal/shedverbs/`, as the shed layer's Go driver: its settle loop, rule table and hand-back, the tackler, and the `shed` config module, told its paths and seams;
  - `internal/loomcli/`'s line says it mounts that driver for `lyx loom drive`;
  - the loom bootstrap paragraph names `internal/sheddriver` for the detached Go driver;
  - the loom config paragraph says the four driver and tackler keys of `loom.yaml`, and of `darn.yaml`, override `shed.yaml`'s defaults, empty, 0 or null inheriting;
  - the batten paragraph says `lyx batten run` drives its own run through the same driver body in-process, with a tackler in the prime's session and the three tackler keys in `batten.yaml`;
  - the `--child-driver` sentence's "unset, the task's `driver_kind`, `llm` by default" becomes the child recipe's `driver_kind`, then `shed.yaml`'s, `llm` by default.
- `internal/sheddriver/doc.go` becomes the package's full design doc, the design D1 to D9 build:
  - the told-geometry rule and the import bound `PATTERN-shed-driver` pins;
  - the loop over `shedverbs`' `--until-stop` and goto bodies;
  - the rule table's rows in order, the repair cap and the settle records at `shedrun.SettleRecords`;
  - the seams a recipe fills and what each one left unwired removes, with D2's bound;
  - the hand-back's report, notices and park, and the tackler's attempts, verdicts, stencil and recipe line;
  - the `shed` config module and how a module's override is resolved over it, with `loomengine.RecipeOverride` as the picker by recipe.
- `internal/loomcli/doc.go`'s `drive` description says the verb mounts `sheddriver` with loom's seams and serves darn runs, and the arbiter paragraph says the driver reaches the arbiter through its arbiter seam.
- `internal/battenshed/doc.go`'s run sentence says `lyx batten run` steps the lifecycle through the Go driver, so an unexpected stop meets the rule table and a cold tackler before the orch, and that the driver lock refuses a second run for the slug.

Every sentence follows semantic line breaks, and the `docs/` links resolve (`PATTERN-markdown-link-integrity`).
