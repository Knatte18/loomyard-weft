# Card 18 — batten-drops-publish-retry

**Delete:**
- `internal/battenshed/publishretry.go`
- `internal/battenshed/innerrun_publishretry_test.go`
- `internal/battencli/publishretry.go`
- `internal/battencli/publishretry_test.go`
- `internal/sheddriver#GoVerifyRetrySpent`
- `internal/battenshed#ChildPublishFailure`

**Edit:**
- `internal/battenshed#InnerRunDeps`
- `internal/battenshed#innerRunProducer`
- `internal/battenshed#NewInnerRun`
- `internal/battenshed#innerRunProducer.Call`
- `internal/battenshed#childWait`
- `internal/battenshed#innerRunProducer.check`
- `internal/battenshed#innerRunProducer.noticeCondition`
- `internal/battenshed#innerRunProducer.binaryRestep`
- `internal/battenshed#runMarkerSuffixes`
- `internal/battencli#battenCLI.wire`
- `internal/testkit/envkit#nilLegal`
- `internal/testkit/envkit#FullEnv`
- `internal/loomcli#intentionallyNil`
- `internal/battencli#intentionallyNil`
- `internal/battenshed#restepHarness.producer`
- `internal/sheddriver#TestSettleRecords`
- `internal/battenshed#TestInnerRun_HaltedStatesWaitExemptWithoutSpawning`

**Extends:**
- `internal/battenshed/doc.go`

**Intent:** Remove batten's publish retry (D11): `publishretry.go`, its `<row>-publish-retry` marker and its settle-record check.
`InnerRunDeps` drops `VerifyRows`, `PublishFailure`, `ChildHead`, `Goto` and `GoVerifyRetrySpent`, and `ChildPublishFailure` goes with `RecordMatchesHalt` and `PublishRetrySpent` in the deleted file.
`innerRunProducer` drops `retryOn`, `childWait` drops `retry` and the three `publishRetry*` fields, `NewInnerRun` and `Call` stop deciding a retry, `check` stops running its step, `noticeCondition` stops suppressing a halt the retry would take and stops adding its text, and `binaryRestep` stops waiting on it.
So a run halted at Merge-In or Finalize gets one state-changed notice like every other halt, and no goto (D11's acceptance: batten sends a notice for a run halted at Merge-In or Finalize, with no goto).
`runMarkerSuffixes` drops the retired suffix.
`battencli`'s wire stops filling the dropped seams and its retry readers go; `sheddriver.GoVerifyRetrySpent`, whose callers were the retry and its reader, goes.
`envkit`, loomcli's and battencli's `intentionallyNil` and the re-step harness's producer drop the seams.
`shedrun.SettleVerifyRetry` stays, so a settle record an older driver wrote with that action still decodes (`PATTERN-tolerant-records`); no code writes or acts on it any more.
`TestInnerRun_HaltedStatesWaitExemptWithoutSpawning` gains rows for a child blocked at Merge-In and at Finalize, each asserting one state-changed notice and no goto.

`doc.go` drops the publish-retry section.

**ImpactSummary:** Batten's halted arm loses one automatic step; its seams were filled only by battencli's wire and the test kit, all listed.
