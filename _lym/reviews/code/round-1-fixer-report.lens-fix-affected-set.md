```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun each finding's set search (F1 flag-name sites, F2 Decode callers, F3 stale `BoardDir`/`_board` sites in docs/overview.md).

No findings.

Findings I left unchecked: none.

- F1: no `--target` (singular) remains in heddle code or docs. The only other flag-name sites, `internal/heddlecli/verbs.go` and the `store.go` messages, now say `--targets`.
- F2: `Decode` is called only from `replayer.go` (base, parent, commit). `decodeFrom` and the `<Bolt checkout>` placeholder are gone from non-test code. The one remaining hit is the test's negative assertion.
- F3: lines 119, 126, 143, 172, 182 and 370 are covered, and the `_board` literal on 119, 126, 143 and 182 is correct because `BoltDirName = "_board"`. Lines 173, 375 and 376 still say `<BoardDir>`. They describe the board's own `board.yaml` and the config layers, which still resolve through `BoardDir`. I did not check them against code, so I report them as outside the finding's list, not as defects.

Calls made:
1. Grep over `internal`, `docs` and `contracts` for singular `--target`, then for `decodeFrom` and `<Bolt checkout>`, then for `_board` and `BoardDir` in `docs/overview.md`. My context had no post-edit repo-wide search results.
