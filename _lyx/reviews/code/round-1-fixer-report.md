# Fixer-report, code round 1

## Changes

- F1 (MEDIUM), fixed as proposed: `internal/loomcli/cli.go` help example now shows `lyx loom circling accept <slug>`.
  The same removed verb sat in the `loom resume` refusal message in `internal/loomcli/resume.go`; it names the live verb too.
  No test asserted the old text.
- F2 (LOW), fixed as proposed: deleted `AlignmentDir` and `AlignmentRel` in `internal/shedrun/paths.go`, the alignment staging and its comment lines in `internal/loomcli/wiring.go`, the `AlignmentDir` line in `internal/shedrun/doc.go`, and the matching rows in `paths_test.go` and `wiring_commitstatus_test.go`.
  `alignmentDirName` and `RunTreeRels.Alignment` stay, since `internal/reportengine/wakes.go` reads archived alignment records through them; the `paths_test.go` expectation builds that path from `RunDirRel` and `alignmentDirName`.
- F3 (LOW), fixed as proposed: added `internal/reportengine/archivedrecords_test.go`, decoding one record of each shape (`request.json`, `delivery.json`, `verdict.json`, `alignment.json`, `attempt.json`, `verdict-<token>.yaml`), the absent-file cases and the error paths.

Gate: `lyx gate test` on `./internal/shedrun`, `./internal/loomcli` and `./internal/reportengine` passed.
One commit per finding.

## Self-check

- fix-consistency: no report findings.
- fix-code-correctness: no findings (one non-finding note: no overflow row for `parentReviewRoundNumber`; unreachable short of a 20-digit directory name).
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
