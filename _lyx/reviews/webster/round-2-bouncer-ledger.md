---
round: 2
ledger:
  - key: retry-landed-slash-command-dropped
    rounds: [1]
    status: resolved
  - key: paste-buffer-leaks-on-failed-paste
    rounds: [1]
    status: resolved
  - key: clihelp-parallel-subtests-share-cobra-tree
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
---

Round 1 raised two LOW design findings, both about `internal/shuttleengine/send.go` and `internal/reedengine/io.go`.
Round 2 does not re-raise either, so both are resolved.

Round 2 raised F1 as `clihelp-parallel-subtests-share-cobra-tree`.
`TestRenderIndex_FormsAndFiltering` in `internal/clihelp/index_test.go` runs parallel subtests over one shared cobra tree.
`visibleLocalFlags` merges flags lazily on first use, which writes `pflag.FlagSet` maps and fails the Publish verify with concurrent map writes.
The fault predates the branch; the fix is to stop sharing the tree across parallel subtests.
