---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
---

# Fixer report, code round 1

## Changes

Commit `3f3d016c6` holds the code changes of both findings.

- F1, `internal/testkit/gofmt_test.go`: `gofmtFinding` now returns a finding when only the file's ending differs,
  naming the line after the formatted form's last line, with the same `lym loom fmt` hint.
  Two table rows cover a missing final newline and extra trailing blank lines.
  The function's doc comment says which line the finding names.
- F2, `_lym/landing/summary.md` (not committed): the "stays whole-tree" bullet now says "outside testdata directories".
  The opening paragraph repeated the claim and now says the same.
- F2, `internal/testkit/gofmt_test.go` and `internal/testkit/doc.go`: the file header, the test's doc comment
  and the package doc now state the testdata exemption.

Gate: `lym gate test ./internal/testkit`, tier0, 11 passed, 0 failed, 0 skipped.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal, LOW (summary's second paragraph claims the guard's reach without the testdata exemption): fixed,
  in `_lym/landing/summary.md`.
- fix-affected-set, LOW (the same sentence): fixed, merged with the fix-proposal finding.

## Disputed

None.

## Deferred

None.

## Rulings

None.
