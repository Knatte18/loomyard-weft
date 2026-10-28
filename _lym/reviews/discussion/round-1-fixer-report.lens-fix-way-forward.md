```yaml
lens: fix-way-forward
outcome: reported
```

Task: check that the ways forward this fix introduced can be carried out and make progress.

Ways forward the fix introduced, and their status:
- D1 acceptance, "`TestConfigTemplate`'s emphasis-line check passes unchanged": this holds. After D1, `board-scope` sits only in `twill-discussion` and `discussion`. No `review-*` or `fix-*` fan names it: `fix-discussion`, `fix-plan` and `fix-code` hold neither `goal-scope` nor `board-fit`. So the test treats it as an emphasis lens, which matches the prose D1 now prescribes.
- D5 acceptance, a `run_settings` or local layer holding `webster_reviewer_fan` or `webster_fan` is admitted: this can be reached. `PrepareRunSettings` runs `module.Migrate` before `checkLocalDocument` (`internal/configreg/runsettings.go`), and loom applies `webster_fan`→`webster_reviewer_fan` (`config.go:1022`) before `webster_reviewer_fan`→`code_reviewer_fan` (`:1035`).
- D6, `speed`'s `darn` and `pawl` blocks set no fan because `code_max_bounces: 0` runs no code round: this matches `internal/runprofile/template.yaml`, where both blocks set `code_max_bounces: 0`.
- Open risk 2, where a fan named only by `discussion_lenses` or `plan_lenses` fails at the writer lead: the operator's existing step still applies, which is to drop the name from the hub fan before the deploy. The risk also says when each kind of failure shows up.

no findings

Tool calls:
- grep of `internal` Go files for `goal-scope` and `board-fit`, to find which tests and fans name them; my context lacked the fan section of the template.
- grep of `internal/lenses/template.yaml` for the two lens names, to get their fan line numbers, for the same reason.
- sed of the template's fan block headers, to map those line numbers to fan names. Two earlier greps returned nothing because of a pattern mismatch.
