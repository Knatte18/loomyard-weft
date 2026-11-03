# Round 1 fixer-report

## Changes

Both findings fixed as proposed, in `internal/heddleengine/shipped.json`, commit 214b6b47.
The two edits share one file and one commit.

- **F1**: the `lym-batching` bullet "Follow the rule above over the Read tool's own advice to read only the part you need." now reads "Read a file whole, or every span you need from it, even though the Read tool advises reading only the part you need." The two "Never" bullets are unchanged.
- **F2**: step 4 of `lym-plan-edits-before-gate` now numbers its four sub-steps `1.` to `4.` under the 3-space indent instead of using `- ` markers.

`lym gate test ./internal/heddleengine` passed: 91 targets, 89 passed, 0 failed, 2 skipped.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

Every fork returned a report.

## Disputed

None.

## Deferred

None.
