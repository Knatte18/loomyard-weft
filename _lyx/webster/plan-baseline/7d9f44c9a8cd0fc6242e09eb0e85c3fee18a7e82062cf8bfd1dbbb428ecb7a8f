# Card 20 — publish-writes-failure

**Edit:**
- `internal/landingshed#Deps`
- `internal/landingshed#verifyGate`
- `internal/landingshed#verifyGate.check`
- `internal/landingshed#verifyGate.checkPublishVerify`
- `internal/landingshed#NewPublish`
- `internal/landingshed#Publish.Call`
- `internal/loomcli#landingDeps`
- `internal/landingshed/publish_verify_test.go`
- `internal/loomcli/landingdeps_verify_test.go`
- `internal/landingshed/doc.go`
- `pattern/PATTERN-verified-tree.md`

**Uses:**
- `plan:internal/verifytree#WritePublishFailure`
- `plan:internal/verifytree#RemovePublishFailure`
- `internal/websterengine#ParseVerifyFailures`

**Intent:**
Have Publish persist its failure (D11).
`landingshed.Deps` gains `FailingTests func(log string) []verifytree.FailedTest`, the told parser; `landingDeps` fills it from `websterengine.ParseVerifyFailures`, keeping the test-kind identities and splitting each id into its import path and test path, so landingshed stays free of websterengine.
`verifyGate` gains an optional failure recorder, which `NewPublish` sets and Finalize's gate leaves unset, so only Publish writes the record.
When `check` (kind plan verify) or `checkPublishVerify` (kind `publish_verify`) sees `StatusFailed`, the recorder writes the record with the failing tests parsed from the log, HEAD, and the merge-in commit; a write failure is logged and never changes the Stuck verdict.
`Publish.Call` hands the recorder the merge-in commit, which is HEAD after a merge-in that was not already up to date and empty otherwise, and removes the record once both verifies pass, before the push.
A dirty tree or a timeout writes no record, since neither names a failing test.
`publish_verify_test.go` gains rows: a failing `publish_verify` writes the record with its kind, tests, log copy, HEAD and merge-in commit; a failing plan verify writes the plan-verify kind; a passing Publish removes a present record; Finalize's failing verify writes none.
`landingdeps_verify_test.go` asserts the wired parser maps a `--- FAIL:` log to its import path and test path.
`internal/landingshed/doc.go` and `pattern/PATTERN-verified-tree.md` state the record's lifecycle.

**ImpactSummary:** Publish's verify gate gains a recorder and Deps a told parser; Publish verdicts are unchanged, and the only new effect is the record file under the verify directory.
