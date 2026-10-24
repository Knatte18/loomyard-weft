# Keep every lens fork report verbatim in the run records

A writer that reviews its work through lenses spawns one fork per lens, and each fork returns a report.
Only a one-line-per-finding summary reached the run records; the full reports survived only in machine-local Claude Code transcripts, which are not in git and do not exist for runs from another machine.
Now every lens fork's final message is saved verbatim in a file of its own beside the record the lens review is written to, so a lens audit reads each report from the archive tags.

## What changes

All four lens fans save their reports:

| Fan | Record | Report |
| --- | --- | --- |
| Plan-Write twill fan | `_lyx/plan/lens-review.md` | `_lyx/plan/lens-review.lens-<lens>.md` |
| Discussion-Write twill fan | the support log | `_lyx/discussion/support-log.lens-<lens>.md` |
| Burler reviewer cluster fan | `round-<n>-review.md` | `round-<n>-review.lens-<lens>.md` |
| Burler fixer self-check fan | `round-<n>-fixer-report.md` | `round-<n>-fixer-report.lens-<lens>.md` |

- New spec `contracts/specs/twill-lens-report-spec.md`, deployed by `lyx stencil` to the specs directory's `twill` family.
  A report is a fenced YAML header (`lens`, and `outcome` of `reported` or `no report`) over the fork's final message, unedited.
  A fork that returned nothing still gets its file, ending at the header.
- The twill directive, the Burler cluster-fan stencil and the fixer self-check stencil tell the writer to save the reports once the last fork has returned and before it judges or edits anything.
  They point at the deployed spec path instead of restating the format.
  A solo review round and a fixer without a fan get unchanged prompts.
- The reviewer seat's and an overlay-scope fixer's write surfaces name the report files, since both stencils otherwise forbid any file but the review or fixer-report.
- New stdlib-only package `internal/lensreport` owns the naming: the prompt-facing path pattern for a record path, and a predicate recognizing a record's report names.
  It is listed in `PATTERN-leaf-packages.md`.
- `burlerengine.Geometry` gains `SpecsDir`, filled by `hubgeom` and `standalonegeom`, so the engine derives no path itself.
- The plan directory treats reports as plan-generation files:
  - `index-file-mismatch` skips `lens-review.lens-*.md`.
  - Plan-Write's stale-plan rotation moves the reports with the other top-level files but never names them in the prior-plan block.
  - webster's plan fingerprint ignores them, as it does `lens-review.md`, so saving a report cannot invalidate a plan.
- Docs: `doc.go` of the touched packages and the twill lines of `docs/overview.md`.

## Design decisions

- A report is named `<record stem>.lens-<lens>.md` and sits where each host's existing commit already stages, so it reaches git with no new commit path.
  The reviewer and fixer fans of one round get distinct names even when they share a lens name, and neither Burler round-file parser matches the new names.
- The writer saves the reports, not Go code: only the writer session sees a fork's final message.
  Saving before judging keeps the text verbatim, before any fix changes what a finding cites.
- No parser or gate reads the reports; their readers are audits.

## Known limits

- A Burler round redone after an archive of its stale outputs overwrites a lens's report instead of archiving it; the earlier text survives only in git history.
- A writer may skip the save; nothing checks that the reports exist.

## To check by hand

- Nothing was run against a live twill or Burler fan; the checks are unit and prompt-rendering tests.
  A real Plan-Write or Burler review run should be read for whether the writer saves one report per lens in the format the spec pins.
