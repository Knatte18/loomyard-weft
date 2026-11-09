```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set. No findings.

**Per change, the set found:**
- **`shedadapters.RubricNotes.RedundancyReport` and the `ReadRubric` fill:**
  - Callers: `internal/shedadapters/bouncer.go:1693` and `:1821` (both through `b.rubricNotes()`), `internal/shedrecipe/entries_burler.go:341` (through `notes`), and the rubric tests.
  - Covered: `bouncer.go` `rubricNotes` and `entries_burler.go:95` both pass the new field.
  - The `notes` struct is built by field name everywhere, so no positional caller breaks.
- **`shedrecipe.Env.RedundancyReport` (new nil-legal seam):**
  - Covered: `envkit.go` `nilLegal` gains the key.
  - `loomcli/wiring.go` sets it, and `wiring_test.go` pins the wired value.
  - `shedrecipe/doc.go` and `loomcli/doc.go` describe it.
- **`BouncerConfig.RedundancyReport`:** covered. It is set in `entries_bouncer.go` and read in `rubricNotes`, and `bouncerprompt_test.go` and `entries_bouncer_test.go` assert it reaches the seed and judge prompts.
- **The rubric marker allowlist:**
  - Covered: `contracts/stencils/rubric_test.go` allowlist gains `redundancy_report`, and the Code-Review rubric is the only stencil carrying it.
  - Not in the diff: `grep` for `review_base` found no other doc or spec listing the rubric markers, so no marker list is left stale.
- **Code-Review rubric stencil:**
  - It gains the test-economy candidate text and a `## Redundancy report` section.
  - The text tells the reviewer to check assertions before filing a fold, delete or keep finding.
  - It does not drop, weaken or contradict an existing rule.
  - Covered against D8.
- **`PATTERN.md` and `pattern/PATTERN-test-speed.md`:**
  - The index line matches the declared text character for character.
  - The background adds `lym gate testmap` and the `redundancy` step to the section on gate runs that take a tier's `env` and a slot.
  - The test-economy section states the package-scoped verdict with the module-wide column, the map, the step and its bounds, and the gate-finding-as-evidence reading.
  - Covered against D9 and the declared change.
- **`docs/overview.md` line on the redundancy step:** updated to say Code-Review reads the report while it matches HEAD. Covered.
- **`gatecli/redundancy_integration_test.go`:** the "only run records changed" row now commits `records/notes.md` outside the `_lym` link. This is a test fixture fix that backs the acceptance criterion about a commit touching only run records. No API or behavior change.
- **Commit messages:** no `Pattern-Departure:` trailer appears, and none is needed.

**Tool calls:**
1. `grep` for `ReadRubric(` and `review_base` across non-test Go, Markdown and YAML files. My context lacked the full set of `ReadRubric` callers and any doc listing the rubric markers.
