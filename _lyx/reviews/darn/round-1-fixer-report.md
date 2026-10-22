## Changes

- F1: `contracts/stencils/orch/orch-template-role.md` now says `lyx board profiles` lists the hub's profiles. Fixed as proposed. Commit c02349c81.
- F2: `internal/runprofile/runprofile_test.go` `TestLoad` gained a `wantDarn` field, set on the "no file" row, asserting the light profile holds darn `{review_max_bounces: 1, review_hard_max_bounces: 2}`. Added as an assertion, not a new test. Fixed as proposed. Commit 04652e7ce. `lyx gate test ./internal/runprofile` passed.
- F3: the stencil bullet now breaks after "before its batten run starts;" with the second clause on a continuation line. Fixed as proposed. F1 and F3 edit adjacent lines of one file and share commit c02349c81.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
