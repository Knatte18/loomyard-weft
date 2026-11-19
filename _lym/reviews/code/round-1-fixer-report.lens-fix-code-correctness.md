```yaml
lens: fix-code-correctness
outcome: reported
```

Lens fix-code-correctness: I traced each changed branch with a concrete input and checked each added test against its defect, with no findings.

No findings.

- **F1:** `findings` now writes the union of the old and pending maps, including the nil-map case. In the new row (attempts 4), turn 2 holds `{A:h1,B:h2}` and arrival 3 passes without a turn. The old overwrite would have dropped A and run a third turn, so the row fails with the defect in place.
- **F9:** With attempts 2, a Pending and a Terminal inner failure leave the failing count at 0, so the pass arrival gets its turn. If either were counted, the check `failing+1 &gt;= 2` would refuse the turn and the row would fail.
- **F4:** `w.rerun.Run(inner)` is nil-safe, because `spawnAndJudgeAgain` already calls it for entries with no mark. Without the fix the new row would see `[false]` instead of `[true]`.
- **F5:** `unrefreshed` lines keep the `pkg: ` prefix that `unrefreshedReason` cuts, so `freshMaps` reads the new reason correctly.
- **F6:** `rewritten` is 2001, so the product is 4,004,001, which is over the cap. The 2001-line middle has no shared prefix or suffix, and the shared line at 1001 sits inside it. Under the LCS path that line would be unmarked, so the row fails with the defect.
- **F7:** The import replaces blank line 8, so the line numbers are unchanged. Each new row fails if its rule regresses: operator rank, the aliased-import exclusion, the local that shadows the logger, or the rune constant.
- **F8:**
  - The rows run in sequence, and the stale-map edit is restored by `t.Cleanup`.
  - The last row commits to the prime after the pair rows, so nothing else is affected.
  - `reasonSharedMapStale` and `reasonNoMapToMutate` are compared by value.
- **F10:** `Parent` is empty only for a root commit. `FirstParentCommits` returns only commits newer than a `since` it has met, and a root commit cannot be newer than any commit. The removed branch was unreachable, and a missing `since` takes the `!met` refusal first.
- **F11:** `readIndex` now uses `os.ReadFile` and `json.Unmarshal`, the same decode `state.ReadJSON` does. The test removes the index lock file after `WriteIndex`, and the reader opens the index through `readSharedTable` when the stamp matches. With the old locking read it would recreate the lock file, so `cmp.Diff` would flag it.
- **F2 and F3:** The F2 assertion fails if the closure returned HEAD as the base, because the output would then carry the `reaches none` line and a `lym gate reach --test` command. The F3 nil checks fail if either assignment is dropped.

Out of scope: the `changedlines_test.go` failure message now prints the row name instead of the inputs, which loses some diagnostic detail for the older rows. It is not a correctness issue.

No further tool calls were made. My context already held every changed file and the tier 0 and tier 1 pass results.
