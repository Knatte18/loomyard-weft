---
round: 1
focus:
  - "internal/shellguard/rows.go `shellScriptReachPattern`, `singleQuotedScriptUnitPattern`, `doubleQuotedScriptUnitPattern`: settle whether the regexes read a sh/bash -c script as the Goal asks, so the Seen command (a sh -c loop with mktemp -d and rm -rf of a variable path) is refused by the computed-rm row; check quote and JSON-escape handling, nested quotes and backtracking or ERE portability for grep -E."
  - "`commandPositionPattern` rewrite: settle whether top-level matching is unchanged for every row that uses it, including the heredoc-line separator branch now in `topLevelReachPattern`, and whether the new do/then/else/if/elif/while/until keywords widen denies beyond what the Goal licenses (false denies on argument text such as `echo do rm $x`)."
  - "`redirectWritePattern` and the new fourth alternative of `rawGoCommandPattern`: settle whether the script-aware branches introduce false denies or misses, e.g. a fd duplication or a quoted target in a script, and whether adding `'` to the word-end class of the go test/build/vet alternative changes existing top-level behaviour."
  - "internal/shellguard/rows_integration_test.go: settle whether the changed `behind_bash_dash_c` expectation (false to true) is a legitimate behaviour change the Goal licenses, and whether each new case adds coverage rather than duplicating one; check that every case's expected value is independent of the pattern code."
  - "Scope: the diff touches `internal/shuttleengine/claudeengine/testdata/settings-golden.json` and its `doc.go`; settle whether those changes follow mechanically from the pattern changes and are licensed by the Goal, and whether the doc comments in `internal/shellguard/doc.go` and `rows.go` still state what the code does, including the claim that the script is read one level deep."
---

Seed focus for round 1. The decision record holds only the board entry under `## Goal`, so the Goal and its amendments are the measuring stick.
