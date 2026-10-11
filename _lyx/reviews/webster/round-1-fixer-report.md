# Webster round 1 — fixer-report

## Changes

- **F1** (MEDIUM, fixed as proposed) — commit `a54a7e7aa`.
  Premise confirmed: `od -c` of `internal/loomengine/template.yaml` ended on `escalation` with no `\n`, and `ConfigTemplate()` returns the file verbatim, so both appended-key rows of `TestLoadConfig_Loads` landed inside the last line's comment.
  Restored the trailing newline.
  Confirmed the `retired fix_start key ignored` row now reaches the parser: with its appended text made malformed (`fix_start: [after-review`), the row failed with `yaml: line 35: did not find expected ',' or ']'`; the probe was reverted before the commit.
  With the valid row, both the `fix_start` and `selfreport` rows load the template defaults and pass.
  No test was added: the finding is not a coverage gap, and the existing rows now test what they name.
- **F2** (NIT, fixed as proposed) — commit `c8174f8bf`.
  Premise confirmed at `internal/shuttleengine/doc.go:73`.
  Dropped the parenthesised `45 seconds` and `20 minutes`; the doc names `holdNoticeSettle` and `holdNoticeMaxDeferral` alone, which `hold.go` declares.

Verify: `lyx gate test ./internal/loomengine ./internal/shuttleengine` is green after both fixes.

## Disputed

None.

## Deferred

None.
