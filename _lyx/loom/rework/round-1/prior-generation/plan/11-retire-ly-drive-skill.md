# Card 11 — Retire the ly-drive skill

**Delete:**
- `plugins/ly/skills/ly-drive/SKILL.md`

**Edit:**
- `plugins/ly/skills/INDEX.md`
- `README.md`
- `docs/overview.md`
- `docs/reference/claude-trust-dialog-repro.md`
- `contracts/specs/refusal-spec.md`
- `contracts/stencils/friction/friction-template-reflection.md`
- `internal/loomengine/template.yaml`
- `internal/loomengine/driver.go`
- `internal/loomcli/cli.go`
- `internal/loomcli/bootstrap.go`
- `internal/loomcli/commitrecords.go`
- `internal/loomcli/driverlaunch.go`
- `internal/loomcli/driverreport.go`
- `internal/loomcli/driverresume.go`
- `internal/loomcli/driverresumeline.go`
- `internal/loomcli/driverspec.go`
- `internal/shedrun/paths.go`
- `internal/shedrun/seed.go`
- `internal/shedcli/seed.go`
- `internal/shedcli/table.go`
- `internal/shedverbs/step.go`
- `internal/battencli/arm.go`
- `internal/reedengine/byname.go`
- `internal/loomcli/driverspec_test.go`
- `internal/loomcli/smoke_bootstrapwiring_test.go`
- `internal/shedadapters/bouncer_seed_test.go`

**Intent:**
Finish D8: with the driver launching from `shed-template-driver` (card 10), the `ly-drive` skill has no user, so it leaves the `ly` plugin and every mention names the driver stencil.

Delete `plugins/ly/skills/ly-drive/SKILL.md`, leaving no `ly-drive/` directory, and drop its row and its "runs only when" line from `plugins/ly/skills/INDEX.md`.
The plugin's version stays `1.0.0`.

Every remaining `ly-drive` mention in code comments, help text, specs, the README and docs is reworded to name the driver stencil (`contracts/stencils/shed/shed-template-driver.md`) or "the loom driver":
the `lyx shed seed` help text in `shedcli/seed.go`, which today says a driver session needs the skill, says the driver session launches from the driver stencil;
`refusal-spec.md` names the driver as its reader;
`friction-template-reflection.md` names the driver's repair and re-step records;
`README.md` drops the skill from the `ly` plugin's list and describes the driver as a stencil-launched session;
`docs/overview.md`'s loom and shed sections describe the driver stencil.
`driverresumeline.go`'s resume line says "Resume the loom driver run" instead of naming the skill.
Test-file comments and the `driverspec_test.go` fixture prompt are reworded the same way; the `bootstrap_test.go` fixture strand named `ly-drive` is an arbitrary strand name, not a mention, and stays.
`docs/research/reed-exploration.md` is a dated research record and stays as written.

Grep check for the reworded help text: no test asserts the old `lyx shed seed` or resume-line wording (`grep -rn "ly-drive" --include=*_test.go` finds only comments and fixtures listed above).
After the card, `grep -rn "ly-drive"` outside `_lyx/`, `.lyx/` and `docs/research/` finds only the `bootstrap_test.go` fixture.

**ImpactSummary:** removes one skill file and rewords comments, help text and docs across loomcli, shedcli, shedrun, shedverbs, battencli, reedengine and the docs; one help text and one resume line change wording, and no test asserts either.

**Commit:** 11: ly: retire the ly-drive skill in favour of the driver stencil
