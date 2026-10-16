# Webster fork and pattern-directive fixes

All twelve batches recorded `status: done`, with no recovery and no stuck batch.

- **01-language-rename-tools** (head `f6da02c43`): `language.yaml` gained `rename_tools`, the plan spec names the rename tool and the departure trailer, and `planparser` classifies symbol Rename pairs and parses the overview's PATTERN changes section.
  Deviations: `internal/planparser/glyphref.go`, `internal/planparser/sections.go`.
- **05-discussion-pattern-changes** (head `89b4036b2`): `discussionparser` checks and reads the record's PATTERN changes section, `AppendDecision` appends a PATTERN change bullet, and `lyx loom decision add` gained the PATTERN-change form.
  Deviations: `internal/discussionparser/append.go`, `append_test.go`, `patternchanges.go`, `validate_test.go`, `internal/loomcli/decision.go`.
- **08-pattern-directive-changes** (head `28a612ccd`): every PATTERN directive gained the optional declared-changes marker.
  Deviations: `internal/pattern/pattern.go`, `pattern_test.go`.
- **09-webster-declared-changes** (head `1be3d8cfb`): the Master and recovery directives render the declared changes.
  No deviations.
- **10-review-loom-declared-changes** (head `a70b9d2c5`): the Burler, Bouncer and plan writers render the declared changes.
  Deviations: `internal/burlerengine/review_test.go`, `internal/loomengine/plan_test.go`, `internal/shedadapters/bouncer_judge_test.go`.
- **11-plan-gate-context-checks** (head `c2731d567`): the plan gates gained `rename-tool-missing` and `pattern-changes-mismatch`.
  Deviations: `internal/loomcli/overlay_test.go`, `parity_test.go`, `validate_test.go`, `internal/loomshed/plancontext.go`, `internal/shedrecipe/fixture_test.go`.
- **12-webster-inline-pack-removal** (head `15938c074`): the inline pack layout is removed and `kickstart_pack_file` retired.
  Deviations: the `card-read-pack`, `pack-card`, `pack-empty` and `pack-unresolved` webster fragment stencils.
- **13-implementer-rename-verify-markers** (head `d4415a490`): the implementer body renders `rename_tools` and `plan_verify`.
  Deviations: `internal/websterengine/render.go`, `template_test.go`.
- **14-rename-tool-refusal** (head `575302d8e`): `begin-batch` and `recover-batch` refuse a symbol Rename with no rename tool.
  No deviations.
- **15-digest-departures** (head `b32fb01d5`): `record-batch` and `recover-batch` carry `Pattern-Departure` trailers in the digest.
  Deviations: `internal/websterengine/git.go`, `gitfake_test.go`, `gitwrap.go`, `recordbatch_test.go`.
- **16-verify-gate-fix-commits** (head `979d03675`): the verify gate writes the fixer's commits and departures into `summary.md`.
  Deviations: `internal/websterengine/summary_test.go`.
  The fork noted that `contracts/specs/webster-spec.md` and `contracts/specs/loom-final-summary-spec.md` still describe the old Merriam-written `## Verify gate fixes` section.
- **17-stencil-wording** (head `d8a64e1a9`): edit directive, Webster-Review rubric, pattern directive and webster stencil wording.
  No deviations.

## Verify gate fixes

The gate's first attempt failed on two tests:
- `TestCLIForm_EveryCommand`: the `Long` help text of `lyx loom decision add` (2021 bytes) and `lyx loom validate-plan` (2188 bytes) exceeded the 2000-byte cap.
- `TestFullEnv_PathFieldsAbsoluteUnderOneRoot`: `FullEnv` left `PatternRecordPath` empty.

The fixer tightened the two help texts in `internal/loomcli/decision.go` and `internal/loomcli/validate.go`, and set `PatternRecordPath` in `internal/testkit/envkit/envkit.go` to the same path as `DecisionRecordPath`.
Its commit is `e60af7b0b` (`fix: shorten validate-plan and decision add help under the cap, set PatternRecordPath in envkit`).
The fixer ran `lyx gate test` and `lyx gate test --tags integration` for the touched packages and `lyx loom lint-comments`, all passing; it did not run the full plan verify.

The scope guard reported only the deviations listed above, all informational.
