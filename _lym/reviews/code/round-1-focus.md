---
round: 1
focus:
  - "internal/fabricengine/pushclassified.go `failRejection`: settle whether the fetch-then-`HasUnpulled` judgment is right on the code, records and bolt sides alike, including the case where the fetch succeeds but the upstream ref is missing or unset, and the case where origin is ahead of HEAD yet HEAD also has unpushed commits."
  - "Decision record D1 says a replay conflict and a destructive refusal stay `not_fast_forward` unjudged, and the classification switch has a separate rejection case at the retry loop (`pushclassified.go` near line 197) and one in `classifyFailure`. Settle whether every path that ends in a bare rejection reaches `failRejection`, and whether any path double-fetches or skips it."
  - "internal/gitrepo/push.go: `PushRebaseFree` now wraps `ErrPushRejected`, where the old comment said a wrapper would break an identity check in `coalesce.go`. Settle whether any remaining caller compares the sentinel by `==` or switch identity, in non-test and test code, and whether `lastLineHolding` still picks the intended line for the auth and rejection triggers."
  - "Acceptance criteria against the diff: D3 (record error quotes git's refusal line, text lacks `remote diverged`), D2 (exactly one notice for the `other` record, error still satisfies `IsPushRejected`), D4 (the change description names the board's pushers and the lock answer, and no lock is added). Settle which of these the diff and the tests actually cover."
  - "Docs the scope lists: `internal/gitrepo/doc.go` is not in the diff, and its text on rejection classes sits near the push-retry paragraphs. Settle whether it still describes the judged class correctly, and whether `internal/loomcli/commitrecords.go` `pushFailureWayForward` still gives right advice now that a rejection can be an `other` failure."
  - "Tests in `pushclasses_integration_test.go`: settle whether the new Bolt row and the declining-hook rows each test distinct behavior, whether the rows that changed from `not_fast_forward` to `other` still pin a real divergence case on the code side, and whether any new test duplicates an existing one."
---

Seed focus only: each entry names where to look and what to settle, and none states a verdict.
