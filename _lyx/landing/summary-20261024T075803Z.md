# Keep every lens fork report verbatim in the run records

A lens fork's final message used to survive only in machine-local Claude Code transcripts.
The lead's one-line entry in `lens-review.md` is too terse for a lens audit, and the transcripts are neither in git nor present for runs from another machine.
Each lens fork's report is now saved verbatim in a support file of its own, beside the file its lens review is recorded in, so audits read it from the archive tags.

## What changes

- All four lens fans save their reports: the Plan-Write and Discussion-Write twill fans (`internal/twilldirective`, hosted by `internal/loomengine`), Burler's reviewer cluster fan and Burler's fixer self-check fan (`internal/burlerengine`).
- A report is named `<record stem>.lens-<lens>.md` and sits next to its record: `_lyx/plan/lens-review.lens-<lens>.md`, `_lyx/discussion/support-log.lens-<lens>.md`, `round-<n>-review.lens-<lens>.md` and `round-<n>-fixer-report.lens-<lens>.md`.
  Each host's existing commit already stages the directory the record sits in, so the reports reach git with no new commit path.
- The format is pinned in the new `contracts/specs/twill-lens-report-spec.md`, registered as `twill-lens-report-spec` and deployed to the specs directory's `twill` family.
  It is a fenced YAML header naming the lens and whether the fork returned a report, over the final message verbatim.
  A fork that returned nothing still gets its file, with the no-report outcome and no body.
- The writer saves every report once the last fork has returned and before it judges or edits, so the text stays verbatim.
  The twill directive's wait step, Burler's consolidation paragraph and Burler's self-check wait step each carry the save, and each points at the deployed spec path instead of restating the format.
- A solo Burler round and a fan-less fixer render the prompts they did before.

## Design decisions

- `internal/lensreport` is a new stdlib-only leaf package that declares the naming once.
  It gives the prompt-facing path pattern for a record path and a predicate that recognizes a record's report names; `pattern/PATTERN-leaf-packages.md` lists it.
- `burlerengine.Geometry` gains `SpecsDir`, filled by `hubgeom` and `standalonegeom` from the specs directory each already knows, so the engine derives no path (PATTERN-told-geometry).
  Tests pin the fill in both builders and its forwarding through `Engine.Review` and `Engine.Fix`.
- The reports belong to the plan generation whose lens review they back:
  - `planparser`'s `index-file-mismatch` skips `lens-review.lens-*.md`.
  - Plan-Write's stale-plan rotation moves the reports with the other top-level files but does not name them in the prior-plan block.
  - `internal/websterengine`'s plan fingerprint and per-file hashes skip them, as they already skip `lens-review.md`.
- The reviewer seat's write surface names the lens reports of a cluster round, and an overlay-scope fixer's names those of a fixer fan.
  The shared reviewer template stays free of the lens-report allowance, so only the fan part carries it.
- Docs updated: `doc.go` of the touched packages and the twill lines of `docs/overview.md`.

## Known limits

- Nothing parses or gates on the reports, and no check catches a writer that skips the save.
- A Burler round redone after its stale outputs are archived overwrites a lens's report rather than archiving it; the earlier text survives only in git history.

## Check by hand

No findings are left open.
A live twill fan and a fanned Burler round were not run end to end; confirm in one that the report files appear beside the record and are committed.
