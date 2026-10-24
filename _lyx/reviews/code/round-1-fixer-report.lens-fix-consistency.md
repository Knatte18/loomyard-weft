```yaml
lens: fix-consistency
outcome: reported
```

Task: check the two rows added to `TestRecoveryRetry` for agreement with their surroundings, under the fix-consistency lens.

**Findings**

1. **LOW** — the test's doc comment at `internal/websterengine/recoverbatch_test.go:1566-1569`, which the fix left unchanged.
   - Summary: the comment no longer lists every false case the table pins.
   - Evidence: it names the false outcomes as "no commits of its own, a fork's earlier commits, a record with no start, a HEAD a reset moved off the start, a range of merge-ins only, the cap, and a stuck or failed recovery". The new rows "a fork record a batch-start reset cleared does not retry" and "a non-terminal recovery a batch-start reset cleared does not retry" pin a record that is not a recovery and a record that is not terminal, and neither is named.
   - The comment's first sentence says "a terminal dead recovery retries at no counted recovery". The rows show that the zero-count shortcut also requires the kind and terminal checks, and the comment doesn't say so.
   - Neither new row is wrong. The comment is a sibling that now understates the table.
   - One clause such as "a non-recovery or non-terminal record a reset cleared" would fix it.

Everything else agrees:
- `RecoveryRetry` (`recoverbatch.go:425-430`) checks kind and terminal before `recoveries == 0`, so both new rows give `(false, 0)` and match `wantRetry` false and `wantRecoveries` 0.
- Neither row sets `wantWarn`, and neither reaches `committedSince`, so no warning appears.
- The row names and the setup mutations (`Kind = "fork"`, `Terminal = false`) match the review's `fix` key.
- The rows sit beside the zero-count true row, in the same table and with the same `deadRecovery` helper.

Further tool calls: none.
