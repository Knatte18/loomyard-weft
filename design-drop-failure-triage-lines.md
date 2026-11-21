# Remove the coverage-map triage lines from failure reports

`drop-failure-triage-lines` · Task · enhancement, gate, webster, stencil · starting · Seed-Child

## Why

The measurement in `docs/research/speed/testmap-use-measure-2026-11-21/report.md` (4c031f8b) found the failure-report triage line "reaches none of this run's changes" wrong in all 6 cases it showed:
each failed test failed because of the run's own change, reached through a tree text scan, an embedded file or a nil seam the coverage maps cannot see.
No fixer used a triage line. The operator decided (2026-11-21) to remove triage lines, keep the reach turn and the mutation step as they are, and invest no more in the maps.

## Change

- Remove the triage lines from failure reports: `internal/gatecli/triage.go` and every caller and consumer (webster, loomshed, runplan verify reports and their stencils and specs).
- Keep the reach note, the reach turn and the mutation step untouched.
- Docs and specs that describe triage lines lose that text.
