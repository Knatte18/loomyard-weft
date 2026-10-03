---
round: 2
ledger:
  - key: merge-untracked-files-reads-weft
    rounds: [1]
    status: resolved
  - key: verify-record-names-pre-run-tree
    rounds: [1]
    status: resolved
  - key: dirtypaths-porcelain-v1-mangles-paths
    rounds: [1]
    status: resolved
  - key: stale-integration-stage-prose
    rounds: [1]
    status: resolved
  - key: comment-line-break-convention
    rounds: [1, 2]
    status: resolved
  - key: verify-gate-attempts-zero-comment
    rounds: [1]
    status: resolved
  - key: fixed-column-wrap-in-new-comments
    rounds: [2]
    status: resolved
  - key: markdown-semicolon-line-breaks
    rounds: [2]
    status: resolved
---

Round 1 raised six findings on cards 1–20;
the fixer resolved the four code findings and the `verify_gate_attempts` comment, ending at `44214954f`.
The comment line-break convention stayed open into round 2 because seven new Go comments still joined independent clauses with a semicolon.

Round 2 approved the code and raised three line-break findings.
They were the remaining semicolon sites, fixed-column wrapping in wholly new comment blocks, and semicolon-joined clauses in new markdown lines.
The fixer resolved each in its own commit, ending at `d97942bcb`, and build, vet and unit tests pass there.

Every entry is resolved.
Edits inside legacy fixed-column sentences that run on into untouched lines stay excluded, as do pre-existing semicolons on token-edited lines and `, so` result tails.
