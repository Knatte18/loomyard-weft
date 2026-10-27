# Fixer-report — round 1

## Changes

- F1 (LOW): fixed as proposed in `55b8abc3`.
  `TestCommitWeftAt_SeedFailureFailsBeforeStaging` in `internal/fabricengine/commitweftat_test.go` replaces `.git/info/exclude` with a directory so the seed fails.
  It asserts a non-nil error, `committed=false`, an empty sha, and the dirty file still untracked.
  The exclude file is restored before the status read, because git itself exits 128 on a directory exclude file.
- Self-check follow-up (`3ede23c6`): the test requires the error text to contain `seed weft artifact excludes`, and the file header lists the seed-failure case.
- Gate repair: the file header and the new test's doc comment were re-broken at sentence boundaries, one sentence per line, replacing fixed-column wraps.

## Self-check

- fix-code-correctness, MEDIUM: the test might pass without the seed branch, since git may fail on the directory exclude file by itself. Fixed with my own change: the error-text assertion, not the proposed `.git/info` regular file.
- fix-consistency, LOW: the file header omitted the new case. Fixed as proposed.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
