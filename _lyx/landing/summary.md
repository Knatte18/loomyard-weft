# Keep every lens fork report verbatim beside its lens review

A lens review used to keep only the writer's one-line dispositions.
Each fork's full report survived only in the machine-local Claude Code transcripts, so an audit of the lenses (which overlap, which findings are real) could not read them from the run records.
Now every lens fan's writer saves each fork's final message, verbatim, in a lens report of its own beside the file its lens review is recorded in.

## What changes

- **Where the reports go.**
  A report is `<record stem>.lens-<lens>.md`, next to the record:
  `_lyx/plan/lens-review.lens-<lens>.md` for Plan-Write, `_lyx/discussion/support-log.lens-<lens>.md` for Discussion-Write, `round-<n>-review.lens-<lens>.md` for Burler's reviewer fan and `round-<n>-fixer-report.lens-<lens>.md` for Burler's fixer self-check fan.
  Each host's existing commit already stages the record's directory, so the reports reach git and the archive tags with no new commit path.
- **The format.**
  The new `contracts/specs/twill-lens-report-spec.md`, registered in `contracts/specs` and deployed under the specs directory's `twill` family, pins it: a fenced YAML block with `lens` and `outcome` (`reported` or `no report`), then the fork's final message unedited.
  A fork that returned nothing still gets a file, so an audit sees the gap.
- **The naming.**
  The new stdlib-only `internal/lensreport` declares the rule: `PathPattern` renders it for a prompt, `IsReportName` recognizes a record's reports, and `SpecName` names the spec.
- **The prompts.**
  The twill directive's wait step (`twill-directive.md`), Burler's consolidation paragraph (`burler-cluster-fan.md`) and the fixer's self-check wait step (`burler-fix-self-check.md`) each tell the writer to save every report once the last fork has returned and before judging.
  Each points at the deployed spec rather than restating it.
  The resume rules in `twill-relaunch.md`, `twill-writer-fork.md` and the self-check now resume from the save step, so a writer that lost its forks still records what it holds.
  `twilldirective.Values` gains `LensReportPath` and `LensReportSpecPath`, both required, and the loom hosts fill them from the told specs directory.
- **Write surfaces.**
  The reviewer seat's write surface names a cluster round's lens reports, and an overlay-scope fixer with a fixer fan has the self-check's lens reports in its write surface.
- **Burler geometry.**
  `burlerengine.Geometry` gains `SpecsDir`, filled by `hubgeom` from the hub's specs directory and by `standalonegeom` from the standalone one, and the review and fix prompt composers take it.
- **The plan directory.**
  `planparser.IsLensReportFileName` recognizes the plan's reports: `index-file-mismatch` skips them, and Plan-Write's stale-plan rotation moves them with their generation but never names them in the prior-plan block.

Docs: `internal/lensreport`'s package doc, `internal/twilldirective` and `internal/burlerengine` `doc.go`, `docs/overview.md`'s Plan-Write lens lines, and a `PATTERN-leaf-packages` section for the new package.

## Design decisions

- **Name a report from its record's stem.**
  A per-directory `lens-report-<lens>.md` would collide when a round's reviewer and fixer fans share a lens name.
  The stem form is also invisible to Burler's round-file parsers, which need the exact `-review.md` or `-<kind>` shape.
- **Flat files, not a subdirectory.**
  Plan-Write's rotation moves only top-level files, so a subdirectory would outlive its generation.
- **The writer writes the reports, not Go.**
  Only the writer session sees a fork's final message, and reading provider transcripts would tie the feature to one engine.
- **Save before judging.**
  The text stays verbatim, ahead of any fix that changes what a finding cites.
- **The specs directory reaches Burler through its geometry, not a constructor parameter,** per PATTERN-told-geometry: both geometry builders already know it.
- **`loomshed` reaches the plan-report predicate through `planparser`,** which already declares `lens-review.md`, so `loomshed`'s told-geometry import allowlist stays as it is.

## Not done

- No gate checks that the reports exist; a writer that skips the save is not caught.
- A Burler round that is redone after its stale outputs are archived overwrites a lens's report instead of archiving it; the earlier text survives only in git history.
