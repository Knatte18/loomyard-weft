# Card 3 — recover-batch and reset --to report-head resolve head_sha

**Edit:**
- `internal/websterengine#awaitTerminal`
- `internal/websterengine#reportHeadSHA`
- `internal/websterengine/recoverbatch_test.go`
- `internal/websterengine/reset_test.go`
- `internal/websterengine/reset_integration_test.go`
- `internal/websterengine/doc.go`
- `contracts/specs/refusal-spec.md`

**Uses:**
- `internal/websterengine#resolveReportHead`
- `internal/websterengine#ErrHeadSHAUnresolved`

**Intent:**
D1's other two consume sites take card 2's `resolveReportHead`, so recover-batch and `reset --to report-head` accept an abbreviated `head_sha` the way record-batch does.

- `awaitTerminal`: once `PollUntilTerminal` returns a terminal digest with a non-empty `HeadSHA`, and after the mid-merge refusal, the digest's `HeadSHA` is replaced by `resolveReportHead`'s answer over the batch's report path before `reconcileReportHead` runs.
  The returned `RecoverResult.Digest` carries the resolved value, so `PersistRecoveryTerminal`'s suspect check, delta end and `CardSHAs` see the full SHA.
  A resolution error is returned as it stands; the strand removal before it is unchanged.
- `reportHeadSHA`: the parsed report's `head_sha` is resolved before the `SHAExists` check.
  An `ErrHeadSHAUnresolved` error is refused as a `reset --to report-head` refusal carrying the resolver's text and way forward.
  A non-hex value is left as given and still meets the missing-commit refusal; a resolved value then meets the descends-from-start check as a full SHA does.

Tests:
- `recoverbatch_test.go`, in `TestRecoverBatch_SecondCall`: a recovery report whose `head_sha` is a 9-digit abbreviation of HEAD records the batch with the digest's `HeadSHA` and `CardSHAs` holding the full SHA, and one naming no commit is refused with `ErrHeadSHAUnresolved`.
  The existing row whose report says `head_sha: deadbeef`, a short hex value that now meets the new refusal, takes a non-hex value so it still pins the mismatch refusal.
- `reset_test.go`, in `TestPlanReset_Resolution`: report-head with an abbreviation naming no commit, and with one naming two commits, each refused with the `git rev-parse HEAD` way forward; a non-hex `head_sha` still meets the missing-commit refusal.
- `reset_integration_test.go`, in `TestPlanReset`: report-head over a report whose `head_sha` is a 9-digit abbreviation resets to the full commit it names.

Docs: `doc.go`'s reset section says report-head resolves an abbreviated `head_sha` as record-batch does.
`refusal-spec.md`'s `report head_sha unresolved` row names record-batch, recover-batch and `reset --to report-head`.

**ImpactSummary:** recover-batch's terminal digest and reset's report-head target hold full SHAs; a short-hex `head_sha` naming no commit moves from the mismatch or missing-commit refusal to D1's.

**Commit:** 3: webster recover-batch and reset --to report-head: resolve an abbreviated head_sha (#456)
