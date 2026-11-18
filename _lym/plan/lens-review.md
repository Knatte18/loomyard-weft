# Lens review

Every lens fork returned a report: coverage, row-validity, forced-edits, guard-reach and way-forward.
Duplicates are merged and filed once under the first lens that raised them.

- coverage, MEDIUM, card 3 runs sidecar and re-add warnings never joined by a row: fixed as proposed; card 3 adds the row `blockRunsSidecar` with `addUnlessErr`, asserting both warnings joined by `; `.
- coverage, MEDIUM, card 3 no row for an unreadable self status: fixed as proposed; card 3 adds the `selfStatusGarbage` field and row (replace, no `add-unless`, no status sidecar).
- coverage, MEDIUM, card 2 dropped the non-runs strand placement row: fixed as proposed; `TestEnsureStatusStrandOn` gains the orch-strand-only row expecting the overview display, and card 2 no longer claims card 1 carries that case.
- coverage, MEDIUM (also row-validity, MEDIUM), card 4 frame section order and blank lines untested: fixed as proposed; card 4 adds a running row with orch state and a fixed dobby line asserting dobby, `""`, self-run line, `""`, orch line, and a done row asserting the rest renders as before.
- coverage, LOW, card 2 seam exits unreached: fixed as proposed for the keep path, the replace failure and the build error, each a new row, with `replaceErr` and `buildErr` named on the fake and seam; the `Status` and executable error returns stay untested as propagated raw errors, which `refusal-spec.md` exempts.
- coverage, LOW, every row seeds pawl, so a hard-coded `pawl` passes: fixed as proposed; card 1 adds a beside-runs row seeded with `shedrun.RecipeTapestry` expecting `Window` `tapestry`.
- coverage, LOW, absent-self-run criterion carried only by accident: fixed as proposed; card 3 names the existing row "a loom status strand holding the overview is replaced by the runs strand" as that case and keeps it without `selfState`.
- row-validity, LOW, new card 3 rows did not state guid `r1` that the loop checks: fixed as proposed; the new rows use `{GUID: "r1", Name: "loom-status", Live: true}`.
- row-validity, LOW, card 3 names no self-status writer: fixed as proposed; the row loop inlines `json.Marshal`, `os.MkdirAll` and `os.WriteFile`.
- row-validity, LOW, deleting `TestStatusStrandCmd` drops the pwsh row: fixed as proposed; card 1 creates `plan:internal/loomengine#statusStrandCmd` taking a `shell.Shell` and `plan:internal/loomengine#TestStatusStrandCmd` carrying both shells' rows.
- row-validity, LOW (also forced-edits, LOW item 4), stale headers of `sharedbootstrap_test.go` and `smoke_drivingsurface_test.go`: fixed as proposed; card 2's Intent names both header rewrites.
- forced-edits, LOW, `fakeAdd` needs an unless-name field: fixed as proposed; card 3 lists `internal/orchcli#fakeAdd` under Edit and gives it an `unless string` field.
- forced-edits, LOW, `ensureRunsStrand`'s doc comment turns false: fixed as proposed; card 3's Intent says the comment is rewritten.
- forced-edits, LOW, `LoomStatusStrandName`'s doc comment goes stale: fixed as proposed; card 1 lists `internal/loomengine#LoomStatusStrandName` under Edit with the rewrite in its Intent, and gains an ImpactSummary.
- forced-edits, LOW, `overview.go` file header and `overviewCmd`'s `Short` go stale: fixed as proposed; card 4's Intent names both rewrites.
- guard-reach, LOW, `AddStrandUnless` matches only the full name, not the legacy bare name: fixed as proposed; cards 2 and 3 word the skip as a strand holding the full `loom-status` name, and card 2 notes a legacy-named strand is kept or replaced before the add.
- way-forward, LOW, `lym loom start` named on a halted run also resumes it: fixed as proposed; card 3's Intent says so.
- way-forward, LOW, the replace-failure warning names `lym loom start` even for a done run: disputed, every fix worse; D1 in `_lym/discussion/decision-record.md` reads "On this branch a failed `ReplaceStrand` returns a warning naming `lym loom start` as well as `lym beam start`", unconditionally, and the only fix, gating the verb on the self run's state, departs from that decision with no Added after Discussion entry recording it, as the lens itself notes.
