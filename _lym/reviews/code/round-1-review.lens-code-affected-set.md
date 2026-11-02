```yaml
lens: code-affected-set
outcome: reported
```

**Task:** the code-affected-set lens, covering what each change forces outside the lines it touches. Per change, the set I found:

- **`ensureRunsStrand` now returns a string (was no value).** The only caller is `internal/orchcli/start.go:483`, and it now appends the warning to `report.Warnings`. Covered.
- **`BeamUp` report warnings are now fed by the runs strand.** The consumers are `beamcli/start.go:30` and `adoptcli/adopt.go:91`, which both pass `report.Warnings` through, and `pawlwiring.go:102`, now surfaced. `BeamUpWorktree` callers (`battencli/cli.go:590`, `adoptcli/green.go:246`, `pawlwiring.go:122`) only treat the dobby fields as errors, so the added warnings change nothing there. Covered.
- **`runsStrandName` renamed to `RunsStrandName`.** No old identifier is left. The other `"runs"` hits are the unrelated JSON keys and temp dirs, plus the test literals in `orchcli/start_test.go` and `cli_test.go`. Covered.
- **`"loom-status"` renamed to `loomengine.LoomStatusStrandName`.** The production literal now lives only in `loomengine/driver.go:28`. The `"loom-status"` hits in `loomcli/bootstrap_test.go:247` and `orchcli/start_test.go:326` are test fixtures, not leftovers. Covered.
- **`statusStrandAddSpec` takes a strands argument.** The callers are `sharedbootstrap.go:279`, which passes `statusResult.Strands`, `bootstrap_test.go` and two calls in `smoke_drivingsurface_test.go`, which pass `nil`. All updated. Covered.
- **New import of `loomengine` into `orchcli/runs.go`.** I found no import-allowlist test covering `orchcli`, and `pattern/PATTERN-leaf-packages.md` doesn't name `orchcli` or `loomengine`. Covered.
- **Package docs.** `internal/loomcli/doc.go` and `internal/orchcli/doc.go` are updated, and `reedengine/doc.go` says nothing that the change falsifies. Covered.
- **`docs/overview.md`.** Not updated, so not covered (finding F1).
- **Stencils, `PATTERN.md` and `pattern/`.** I found no stencil text on either strand, and the diff declares no PATTERN change. No entry is violated: `PATTERN-dobby-orch-pair` is untouched because `BeamUp` is still the only bring-up path. Covered.
- **Commit messages and `Pattern-Departure:`.** Neither commit carries a `Pattern-Departure:` trailer, and none is needed.

**F1: LOW, consistency.** `docs/overview.md` still describes the old overview-slot behaviour. Locations:
- `docs/overview.md:539` and `:811` say the loom status strand is the session's overview window. It now goes to a window of its own with `KeepViewer` when a beam's `runs` strand holds the overview.
- `docs/overview.md:421` and `:677` say `lym beam start` ensures the `runs` strand. They don't say it replaces a `loom-status` holder, or that a failure now becomes a report warning.
- Evidence: `internal/loomcli/bootstrap.go:26-39`, `internal/orchcli/runs.go:48-107`, and `internal/loomcli/doc.go:53`, which was updated.
- Fix: update those four passages in `docs/overview.md` to match the updated `doc.go` text.

I did not read the repository's constraints document to see whether it names `docs/overview.md` as required for this kind of change, so LOW is a floor. It is a candidate for MEDIUM if that document requires it.

**Tool calls (3 of 6):**
1. Grep of `*.md`, `*.yaml`, `*.json` and `*.tmpl` for the strand names. My context had no doc search. It found the overview doc hits and no stencil hits.
2. Grep for import-rule tests and `orchcli`'s `loomengine` imports. My context lacked the new import's constraints. It found no allowlist and one new import.
3. Grep for `BeamUp` and `BeamUpWorktree` callers and `Warnings` consumers. My context had no caller list for the warnings. It showed the pass-through is covered.
4. A final grep (the fourth call, so 4 of 6, not 3) covered `PATTERN-leaf-packages` and the overview docs. My context had not read `docs/overview.md`. It produced F1.
