---
round: 1
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
    rounds: [1]
    status: open
  - key: verify-gate-attempts-zero-comment
    rounds: [1]
    status: resolved
---

Round 1 reviewed `5805c58fb..efdf79946`, cards 1–20, and raised one MEDIUM, three LOW and two NIT findings.
The fixer addressed each in its own commit, ending at `44214954f`, and build, vet, unit and integration tests pass there.

Resolved:

- `MergeUntrackedFiles` lists the warp only.
- `Verify` records a pass only when HEAD's tree is unchanged after the run.
- `DirtyPaths` parses `--porcelain -z` verbatim.
- The retired integration-fork, integration-report and `ParseIntegrationReport` prose is reworded.
- The `verify_gate_attempts` zero comment is corrected.

Still open: the comment line-break convention.
Every site the review listed was fixed, but seven new or changed Go comments the list missed still join independent clauses with a semicolon.
They are `fabricengine/mergestage.go:131`, `fabricengine/mutation.go:83`, `websterengine/cardgate.go:2`, `websterengine/verifygatereport.go:64`, `websterengine/doc.go:401`, `websterengine/runlevel.go:724` and `websterengine/runlevel_test.go:113`.
Edits inside legacy fixed-column paragraphs are excluded, because fixing them would rewrap untouched lines.
