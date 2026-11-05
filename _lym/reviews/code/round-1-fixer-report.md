# Fixer report — code round 1

## Changes

- **F1 (LOW), fixed as proposed.**
  - `internal/pattern/doc.go`: after the `(test)` definition, added that on a cluster entry `(test)` means at least one member rule is test-enforced, and that the cluster's background file marks each review-only member rule.
  - Added the line "Held by review alone; no test enforces it." under every `Rule:` line without `(test)` in the background files of the clusters that carry `(test)`: `PATTERN-pair-remote`, `PATTERN-reed-panes`, `PATTERN-daemon-wakeups`, `PATTERN-agent-addressing`, `PATTERN-stencil-ownership`, `PATTERN-run-records`, `PATTERN-lym-files`, `PATTERN-gate-slot` and `PATTERN-fabric-boundary`.
  - `PATTERN.md` is unchanged, as the fix asked.
  - Commit `78a1697f`. `lym gate test ./internal/pattern` passed.
- **Self-check follow-up.** `PATTERN-shed-driver` carries `(test)` and has three review-only members the review's location list did not name; marked them the same way. Separate commit, gate passed again.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, `PATTERN-shed-driver` unmarked members; fixed as proposed.
- fix-proposal: MEDIUM, same `PATTERN-shed-driver` gap (duplicate of the above); fixed as proposed.
- fix-affected-set: MEDIUM, same `PATTERN-shed-driver` gap (duplicate); fixed as proposed. Unchecked: `PATTERN-github-auth` has no background file, so it has no members to mark.

All four forks returned a report.

## Disputed

None.

## Deferred

None.
