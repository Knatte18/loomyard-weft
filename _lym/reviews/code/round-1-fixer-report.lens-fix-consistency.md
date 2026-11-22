```yaml
lens: fix-consistency
outcome: reported
```

No findings.

I looked for twins of the changed text that were left as they were, and found none.

- **"After its tiers" phrasing:** a repo-wide grep (excluding `_lym`, `.lym`, `.scratch`) finds no remaining text that says the report steps run after the tiers.
  - The files that match now agree with the fix: `internal/gateslot/tiers.go:118`, `internal/gatecli/reports.go:2`, `internal/gatecli/doc.go:366`, and `pattern/PATTERN-test-speed.md:226`, `:330` and `:345`.
  - `internal/gateslot/tiers.go:116`, the `Lints` field doc, still says lints run after the tiers, which stays true.
  - `PATTERN-test-speed.md:339`, the mutation step "after its other report steps", is unchanged and consistent.
- **`internal/gatecli/doc.go:213`:** the "acquires its slots after the last listed tier and vet run released theirs" line is about the guard step, not the reports, so it is not a stale twin.
- **Process-cap doc:** `internal/gatecli/doc.go` "# Process cap" gained the sentences about processes outside a running tier command and about lost parentage. They match `countable` in `internal/gatecli/processcap.go` and the new rows in `internal/gatecli/processcap_test.go`.
- **`processCapFor` callers:** the field type in `internal/gatecli/cli.go`, the call in `internal/gatecli/tierrun.go` and the stub in `internal/gatecli/tierrun_test.go` all use the three-argument form.
- **`docs/research/speed/testmap-use-measure-2026-11-21/report.md:55`:** "A call's time after its tiers holds the map refresh…" is a measurement report, which the prose rules exempt from staying current.

Tool calls made (2):
1. `grep` of the working tree for "after its tiers"-style phrases, to find stale twins of the changed text that my context lacked.
2. `sed` and `grep` over `internal/gatecli/doc.go:208-216` and `:355-370`, `internal/gateslot/tiers.go:118` and `pattern/PATTERN-test-speed.md:336-342`, to confirm the neighbouring lines of the hits agree.
