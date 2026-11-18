---
dispositions:
  - id: F1
    disposition: fixed
---

## Changes

- F1: in `internal/editdirective/editdirective_test.go`, deleted the `scratchBuildBan` const and its `strings.Contains` check from `TestDirective_ReadsTheEnginesCopyWithoutBanner`, and restored the doc comment to "its own tool wording is present, with no marker, leading comment or trailing newline."
  The engine tool-wording, marker, leading-comment and trailing-newline assertions are unchanged.
  `lym gate test ./internal/editdirective` passes.
  Committed as 12c343d3.

## Gate findings

The verify run failed tier 1 in `internal/fronttask` and `internal/slackcli`, whose tests all passed.
The failure was tmuxkit's report that `/tmp/.git` appeared during the run and was removed as an empty directory.
The directory came from a test or process running beside those packages: this round's only change deletes an assertion in `internal/editdirective`, which writes nothing.
`/tmp/.git` no longer exists, and `lym gate test --tier 1 ./internal/fronttask ./internal/slackcli` now passes (19 passed, 0 failed).
No file changed for this finding.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
