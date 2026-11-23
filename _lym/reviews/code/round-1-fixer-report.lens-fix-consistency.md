```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens over the round-1 fix change set.

**Findings**

- **LOW:** `internal/shedadapters/bouncerdecision.go`, the doc comment of `settleDecisionAccept`. Its line "At the budget at or past the hard cap the judge outputs are archived and the run halts" is now stale. The F7 change archives `verificationPath(...)` along with `judgeOutputs`, and the comment still names only the judge outputs. The `rejudge` doc comment in `bouncer.go:1213` was updated to name the verification, so this sibling is inconsistent with it. Evidence: `bouncerdecision.go:190-191` against the code at `bouncerdecision.go:210`.

Checked, no finding:
- `doc.go:171` ("each entry it closes is resolved in the ledger") still holds.
- The `blockingEscalationDetail` doc and the `bouncerverify.go` header agree with the new finding-id gate.
- The stencil's output block, its rules and `parseVerification` agree on `finding` and on listing each key once.
- The two judge-run sites at `bouncer.go:2062` and `bouncer.go:2157` only declare outputs and do not archive, so F7 has no twin there.
- No code globs the run directory for `*.tmp`, so the ledger's `.tmp` sibling collides with nothing.
- The only users of `parseVerification` are the two sites in `bouncerverify.go`, and both take the new `map[string]string`.

**Tool calls**
1. `sed` of `bouncer.go:2050-2075` and `2150-2170`, with a grep of `verificationPath` and `verification.md`. I used them to look for twin judge-output archive sites and other references to the verification file, which my context lacked.
2. The same call grepped for `.tmp` globs in `shedadapters`, to confirm the temporary ledger name is not swept up anywhere.
