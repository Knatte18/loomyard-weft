```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the changed code in `outbound.go` and the added tests, by running concrete inputs through them.

**Traced inputs**
- **F1, row 1:**
  - Batch is stale 1, stale 2 and fresh 3, and the fresh post fails.
  - `postMessages` records `Stale=2` and `Handled=[1,2]` on disk, then returns the post error.
  - `reportStale` posts the line, which succeeds. It zeroes the count and writes the record.
  - The second read skips ids 1 and 2, posts id 3 and posts no further line. `texts()` is `[line, fresh]`, which matches `wantLineIndex` 0.
- **F1, row 2:**
  - The line post also fails, so it is logged and `Stale=2` stays in the record.
  - The second read posts id 3, then the line. `texts()` is `[fresh, line]`, which matches index 1.
- **Defect check:** with the old local counter, both rows end with one accepted text, so `len(answers) != 2` fails. Both rows fail without the fix.
- **Budget return:**
  - `postMessages` returns `errReadBudget`, `post` reports any stale count, and `read` still sees `errReadBudget` through `errors.Is`.
  - The budget test has no stale mail, so it is unaffected.
- **Success path:** `read` writes an empty record after the handler returns nil, which clears any leftover count.
- **`reportStale` failure with posts fine:** `post` returns the wrapped error, and the record keeps `Handled` and `Stale`. The redelivered read posts only the line.
- **F4 (both rows):**
  - Five refusals and one try of the replacement line make six `PostMessage` calls, so `callCount() == 6` holds.
  - A nil `replacementFailure` pops as a success.
  - The record is `{}` after the acknowledgement, so the "no refusals" check holds.
- **F2 and F3:** both new tests ran and passed in the tier-1 log. The swapped-worktree case for F3 would invert both rows, so it fails with the defect in place.

**Findings**

- **LOW** — `post` and `reportStale` in `internal/slackengine/outbound.go`: the count line can post twice if the record write fails.
  - **Evidence:** when `postMessages` returns a `writeOutbox` error, `reportStale` still posts the line, and its own `writeOutbox` then probably fails too. The on-disk record keeps the old `Stale`, so the next read posts the line again.
  - The `post` doc comment says the line "survives a failed read", and it does for a Slack failure. It does not post exactly once when the disk write is what failed.
  - This only happens when the disk is failing. The cost is a duplicate line, which fits the relay's at-least-once rule.

- **LOW** — `post` in `internal/slackengine/outbound.go`: after a rate-limited message post, `reportStale` makes another Slack call straight away.
  - **Evidence:** when `postErr` is a rate limit and the record has stale mail, `reportStale` posts the line into a Slack that just answered with a limit. This failure is only logged, and the count stays in the record. `read` still returns the rate-limit wait from `postErr`.
  - The code is correct, but it costs an extra refused call. I did not check whether `isRateLimit` finds a rate limit through the `%w` wrap, because `post` returns `postErr` and not `reportErr` in that case.

No test passes with its defect in place, and no caller of a changed signature is broken. `refused` now takes the `*outboxRecord` that `postMessages` already held, and its only caller is updated.

**Tool calls:** none. My context already held every changed file and the gate results.
