```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: I found nothing the fix left stale.

no findings

- **Accept wording:** The prose twins of the `Accepted` → `AcceptedHead` change still read correctly. These are `finalize.go:620`, `doc.go:96` and the `circling.go` file header, each of which says "accepted" or "marks the record accepted". The `FixerAttempts` field comment and the `judgeFixerChange` doc were updated.
- **F6 rename scope:** `internal/landingshed/handback_test.go:30` still has a `RerouteCapAtFinalize` row. It expects `HandsBack` to answer false, which matches the new rule. It is a negative assertion, so it is consistent and needs no change.
- **Spec and overview:** Both pass the reroute-edge wording check, since `doc.go:159` already says neither landing row has a reroute edge.

Calls (2 of 6):
1. A `grep` for `accepted`, `accepted_head` and reroute-cap wording across `landingshed`, `circling.go`, docs and the spec. Context did not show the sibling prose and tests that mention the old flag.
2. `sed` of `handback_test.go`, `landingshed/doc.go` and the `circling.go` header. Context lacked the hand-back rows and the doc claims about the reroute edge and the accept verb.
