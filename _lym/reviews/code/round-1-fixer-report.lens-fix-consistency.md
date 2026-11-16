```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: I checked the change set against its surrounding text. I found one stale twin, a LOW.

**LOW — `internal/gatecli/doc.go:1154` (the "Bound:" line next to the F2 edit at `:1151-1152`)**
- The bound reads "a contained test's kill beside other runs is not rerun alone, so a disturbed run can refute it falsely."
- The F2 fix keeps a candidate's own kill from the candidate pass, which runs with `rerunFailed` false. The cover pass reruns only a covering test's kill, and that kill is deleted before the rerun.
- The bound is therefore still accurate for the candidate's own kill. The lines right above it now say a covering test that is also a contained test is rerun alone, and the bound doesn't separate the two cases. A reader can take the bound as contradicting `:1151-1152`.
- Fix: reword the bound to "a contained test's own kill beside other runs is not rerun alone", or add "as a candidate".

Everything else I checked agrees with the change set:
- `redundantmutants.go:168-169` and `doc.go:1151-1152` carry the same F2 wording.
- The `redundantCandidate.Covering` comment, `doc.go:1138-1139` and `doc.go:1159` all describe the `CoveringTests` pick.
- The gateslot `template.yaml` and `doc.go` timeout-key edits agree with `redundantmutants.go:35-37` and the `gatecli/doc.go:1144` bound text.
- The README and `docs/overview.md:932-937` now agree on "confirmed by mutation".
- The redundant verb's `Long` text in `redundant.go` makes no covering claim that clashes with the printed pick.
- `docs/overview.md:940` is the testmap bound, which these edits don't change.

Tool calls made:
1. `sed -n 930,950p docs/overview.md` and a grep for "rerun alone" wording across `*.go` and `*.md`. My context lacked the overview text around the verb and any twin phrasing of the rerun rule.
