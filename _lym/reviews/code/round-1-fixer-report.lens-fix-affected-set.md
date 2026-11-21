```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Set checks I ran:
- **F3 (retry-window wording):** a grep for "retried within", "within the same minute" and "a retry clears" finds no stale twin. The remaining mentions are `finalize_pr.go:459` ("retried for up to `prMergeRetryWindow`", which still holds), `mergestate.go:12` and `pullrequest/doc.go:40`, and all agree with the code.
- **F4 (beam wording):** only two sites state "never names cancel / status verb". They are `finalize_pr.go:573` and `doc.go:636`, and both now carry the exception. `beamcli/doc.go:103` describes beam verbs, not the Finalize halt, so it is not affected.
- **F5 (`RefusalOther` contract):** `pullrequest/doc.go:44` says only "any other refusal", which does not contradict the new comment. `client.go:434` is the single classifier site.
- **F6 (`mergeCauses` consumers):** the only non-test callers of `ReadMergeState` are `beamcli/pr.go:350`, `beamcli/land.go:310` and `finalize_pr.go:505`. The summary now names the two beam verbs. `land.go:331` has its own "not the beam's head" text, built separately from `mergeCauses`, so it is not a missed member.
- **F1 (uncomputed-mergeability path):** `MergeabilityUncomputed` is consulted only in `mergeRefusalReason`. The beam path inside it is not covered by a test row. The review rejected a beam row for the resume-only text, but this branch is a separate one. I rate that a gap below LOW and do not report it as a finding.

Tool calls:
1. `grep` for stale retry and beam wording across `.go` and `.md` files: the changed docs' twins were not in my context.
2. `grep` for `ReadMergeState`, `mergeCauses`, `RefusalOther` and `MergeRetryWindow` in non-test files: the full caller set was not in my context.
3. `Read` of `internal/pullrequest/doc.go` lines 36–47: the package doc's `RefusalOther` and `ReadMergeState` text was not in my context.
