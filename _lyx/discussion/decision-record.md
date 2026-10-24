# Decision record — every lens fork report is kept verbatim in the run records

## Goal

Every lens fork's final message is saved verbatim in a support file of its own beside the file its lens review is recorded in, so an audit of the lenses reads each report from the run records instead of from machine-local transcripts.

## Scope

- The four lens fans: the Plan-Write and Discussion-Write twill fans (`internal/twilldirective`, hosted by `internal/loomengine`), Burler's reviewer cluster fan and Burler's fixer self-check fan (`internal/burlerengine`).
- A new format spec, `contracts/specs/twill-lens-report-spec.md`, registered in `contracts/specs` so it deploys beside the other specs.
- A stdlib-only package that owns the lens report file naming, used by every host and by the plan directory's checks.
- The plan directory's index check and Plan-Write's stale-plan rotation learn the new files.
- Docs: the touched packages' `doc.go`, `docs/overview.md`'s twill lines.

Out of scope: any parser or gate that reads the reports; archiving a stale Burler round's lens reports.

## Decisions

### D1: A report sits beside the file its lens review is recorded in, named `<record stem>.lens-<lens>.md`

The record is `lens-review.md` for Plan-Write, the support log for Discussion-Write, the round's review file for the reviewer fan and the round's fixer-report for the fixer fan.
So the reports are `_lyx/plan/lens-review.lens-<lens>.md`, `_lyx/discussion/support-log.lens-<lens>.md`, `round-<n>-review.lens-<lens>.md` and `round-<n>-fixer-report.lens-<lens>.md`.
Rationale: each host's existing commit already stages the directory the record sits in, so the reports reach git and the archive tags with no new commit path;
the reviewer and fixer fans of one round get distinct names even when they share a lens name;
and neither Burler round-file parser matches the new names, since both need the exact `-review.md` or `-<kind>` shape.

### D2: One stdlib-only package, `internal/lensreport`, declares the naming

It exposes the prompt-facing path pattern for a record path and a predicate that tells a record's report file names.
Rationale: four hosts render the pattern and the plan directory's checks recognize it, so the rule is declared once instead of in each.

### D3: The format is a fenced YAML header over the fork's final message, pinned in `twill-lens-report-spec`

The header names the lens and whether the fork returned a report; the body is the final message verbatim, never edited, merged or summarized.
A fork that returned no report still gets its file, with the no-report outcome and no body.
The spec registers as `twill-lens-report-spec`, deployed to the specs directory's `twill` family, and every stencil points at its deployed path instead of restating it.
Rationale: an audit needs both the full text and which forks returned nothing, and PATTERN-producer-pointer-rule wants the instruction to point at the contract.

### D4: The writer saves every report once the last fork has returned and before it judges or edits

The twill directive's wait step, Burler's consolidation paragraph and Burler's self-check wait step each carry the save.
Rationale: saving before judging keeps the text verbatim, before any fix changes what a finding cites.

### D5: Burler's engine is told the specs directory through its geometry

`burlerengine.Geometry` gains `SpecsDir`, filled by `hubgeom` from the hub's specs directory and by `standalonegeom` from its own.
Rationale: PATTERN-told-geometry has the engine derive no path, and both geometry builders already know the specs directory.

### D6: The reports join the writers' write surfaces

The reviewer seat's write surface names the lens reports of a cluster round, and an overlay-scope fixer's names those of a fixer fan.
Rationale: both stencils otherwise forbid any file but the review or the fixer-report.

### D7: The plan directory treats the reports as plan-generation files

`index-file-mismatch` skips a `lens-review.lens-*.md` file, and Plan-Write's rotation moves the reports with the other top-level files but never names them in the prior-plan block.
Rationale: the reports belong to the generation whose lens review they back, as `lens-review.md` does.

## Constraints

- PATTERN-told-geometry: no engine derives the specs directory.
- PATTERN-producer-pointer-rule: stencils point at the spec, never restate its format.
- PATTERN-stencil-ownership: the spec is read from its deployed path at call time.
- PATTERN-plan-generation: the reports rotate with the plan generation.
- PATTERN-comment-line-breaks and semantic line breaks in every touched file.

## Auto-mode assumptions

- A darn run holds no interview; the scope above is read from the board entry and the code.
- "Run records" means the tracked `_lyx` directories each host already commits.
- The writer, not Go, writes the reports: only the writer session sees a fork's final message.

## Open risks

- A Burler round that is redone after an archive of its stale outputs overwrites a lens's report rather than archiving it; the earlier text survives only in git history.
- A writer may skip the save; no gate checks the reports.

## Acceptance criteria

- The rendered Plan-Write and Discussion-Write twill directives name the report path pattern beside their record and the deployed spec path.
- A fanned Burler review prompt and a fixer prompt with a fixer fan name their report path patterns and the spec path; a solo round's and a fan-less fixer's prompts are unchanged.
- `lyx stencil` deploys `twill-lens-report-spec` under the specs directory's `twill` family.
- A plan directory holding `lens-review.lens-<lens>.md` raises no `index-file-mismatch`.
- Plan-Write's rotation moves a lens report and does not announce it.
- `internal/lensreport` maps each record path to its pattern and recognizes exactly its report names.
