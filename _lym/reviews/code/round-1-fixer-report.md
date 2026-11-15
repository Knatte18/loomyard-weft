# Fixer-report, code round 1

## Changes

- **F1** (fixed as proposed), commit `9e755c8a`: `internal/configsync/configsync_test.go`, `TestReconcileHubWideAt_SeedOnlyFileFollowsItsTemplate`.
  - Added row "an unedited file without the reviewer model gains the template's": hub and base are the template without the darn block's `code_reviewer_model`; it expects `light.darn.loom.code_reviewer_model` in Added and the `opus[high]` line in the rewritten file.
  - Added row "an operator's own reviewer model is kept and listed": the hub holds `sonnet[high]` over that base; it expects an empty Added, the key in Differs and the hub's value kept.
  - The Differs text carries the engine's quoted-string rendering, so the expected line quotes both values.
- **F2** (fixed as proposed), commit `44cd8ad4`: `internal/runprofile/runprofile_test.go`.
  - Added a `//testtiming:keep <reason>` line to `TestTemplateLadder` and to `TestTemplateDocsProfile`, each naming what the test pins that no covering test asserts.
  - The marker sits in the doc block after a bare `//` line, directly above `func`, as the existing markers do.

`lym gate test ./internal/configsync ./internal/runprofile` passed all 201 targets.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

Every fork returned a report; none is a "no report".

## Disputed

None.

## Deferred

None.

## Rulings

None.
