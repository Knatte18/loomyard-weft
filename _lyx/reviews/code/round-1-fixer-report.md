# Round 1 fixer-report

## Changes

- F1 fixed as proposed, commit `24aa85a5e`.
  In `internal/gitexec/doc.go`, the sentence "The hold only delays the call's start and never fails it." moved from the end of `# Stall bound` to directly after the pause-wait sentence in `# Remote bounds`, above the `# Stall bound` heading.

## Disputed

None.

## Deferred

None.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings; a wording nit ("waits" versus "the hold") was noted by the lens and is not a finding.
- fix-affected-set: no findings.
