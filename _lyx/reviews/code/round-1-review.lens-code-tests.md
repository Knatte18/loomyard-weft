```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens over the amendment, which is the `docsLinkAllowlist` entry in `internal/lyxcwd/docslink_test.go` (commit 015a8b14e). The merge of 9c167c500 adds only `docs/research/` files and no tests.

**Per behaviour, the test that pins it**
- The allowlist entry exempts only its exact (file, target) key.
  - `TestEnforcement_MarkdownLinks/repo` pins it. It scans `docs` with `docsLinkAllowlist` and fails on any non-allowlisted break.
  - The same key-matching mechanism is pinned by the subtest "allowlisted pair produces no break and leaves unmatched empty" (line 504).
  - A different link in the same file, or the same target in another file, is not exempted. Matching is on the whole key, so every other broken link under `docs/` still fails the `repo` subtest.
- The entry goes stale, and the scan reports it deletable, once the link resolves or the file is gone.
  - `TestEnforcement_MarkdownLinks/repo` reports a stale entry through its `stale` loop.
  - The mechanism is pinned by the subtests at lines 518 and 533.
- The allowlist change does not weaken `TestEnforcement_MarkdownLinks` for files outside the research record. The entry is a single key, and nothing else in the scan changed.
- The D3 Location literal scan is outside this diff's test changes, so I have no row to cite for it.

**Test economy:** no new test or row was added, only data in the allowlist var. The mechanism is already covered by the existing subtests above, so nothing is redundant and nothing is uncovered.

**Findings:** none.

Tool calls:
1. `grep -n 't.Run(".*\(allow\|stale\)' internal/lyxcwd/docslink_test.go`. I ran it to confirm the existing allowlist and stale subtests, which the earlier reading had not shown.
