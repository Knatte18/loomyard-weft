---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/slackengine/inbound.go:678
    summary: A forwarded message whose text and file lines are empty now posts an empty body, which creel refuses, so the read retries for five minutes and holds later messages.
    fix: In `forward`, after computing `posted`, return `true, nil` without posting when `strings.TrimSpace(posted) == ""`, logging a debug line like the skip in `handle`. Add one row to the forwarding table in `TestInboundRead` in `inbound_test.go` that says a message with empty text and no files, asserts `wantPosts()` is empty and the cursor advances.
---

### [LOW] Empty forwarded body is no longer masked by the prefix

**Location:** `internal/slackengine/inbound.go:678`
**Issue:** The old body `"[slack] "` was never empty.
The new body is the trimmed text plus file lines, so a message with empty text and no files posts `""`.
`validateBody` in `internal/creelengine/verbs.go:22-26` refuses an empty body, so `retryOrGiveUp` retries the message for the five-minute window, holds the channel's later messages, then tells the operator it was not delivered.
Slack rarely sends such a message, which keeps this at LOW, and no test row covers it.

## Rejected

- Operator text that itself opens with `[slack presence]` or `[slack relay]` is now read as a notice (code-correctness): it is the operator's own wording, the old prefix had the same edge, and the focus asks only that the role file cover every notice the engine sends, which it does.
- Assistant verdict `APPROVED` with no findings: noted as one opinion; it did not examine the empty-body path, which F1 records.

## Checks that held

- No reader keys on the removed prefix: a repo grep for `[slack]` and `forwardPrefix` finds only the `[slack presence]` and `[slack relay]` strings.
- Every `Post` site of the relay (`holder.go`, `keeper.go`, `relay.go`, `outbound.go`, `inbound.go`) keeps `relayNoticePrefix` or `presenceNoticePrefix`, so the role file's "any other mail" rule covers only the operator's words.
- `docs/overview.md` and `internal/slackengine/doc.go` no longer describe the prefix.
- The edited tests assert the bare body, the cap rows sit at 4096 and 4097, and none duplicates another.
- The diff touches no PATTERN entry and the commit needs no `Pattern-Departure:` trailer.
