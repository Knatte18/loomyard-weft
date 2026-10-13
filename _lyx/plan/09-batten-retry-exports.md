# Card 9 — batten-retry-exports

**Rename:**
- `internal/battenshed#recordMatchesHalt` -> `internal/battenshed#RecordMatchesHalt`

**Create:**
- `internal/battenshed#PublishRetrySpent`
- `internal/battenshed#TestPublishRetrySpent`

**Edit:**
- `internal/battenshed#innerRunProducer.readPublishRetryMarker`

**Intent:**
D4's two shared reads, exported from `internal/battenshed` so the Go driver applies batten's own match and reads batten's own retry, never a copy of either.

`recordMatchesHalt` becomes `RecordMatchesHalt`, its body and its one caller in `publishRetryDecision` unchanged but for the name, and its doc comment opening with the new name.
It stays a pure function over `ChildPublishFailure`, a HEAD, a status and the verify rows, inside battenshed's import allowlist.

`PublishRetrySpent` reports whether batten's publish-retry marker at `publishRetryFile(scratchDir, producer)` exists in any state: a started, goto-done, resumed or capped retry each means batten took the run's one retry, so the Go driver retries no more.
An absent marker is false, and a marker that does not decode is the error the producer's own read already returns.
`innerRunProducer.readPublishRetryMarker` delegates to the same package-level decode over the path, so the two reads cannot disagree.
Bound: the read is only as current as batten's marker write, which batten makes before its goto.

Test `TestPublishRetrySpent` in `innerrun_publishretry_test.go`, a table over a temp scratch directory: no marker reads false; a marker in each of the four states reads true; a malformed marker returns an error.
`TestInnerRun_PublishRetryDeclinesRecordsThatDoNotMatchTheHalt` already pins the renamed match through the producer.
The `internal/battenshed` row of `test-budget.yaml` already has room for the one new test.

**ImpactSummary:** one rename with a single in-package caller and one new read sharing the producer's decode; battenshed's behavior is unchanged.
