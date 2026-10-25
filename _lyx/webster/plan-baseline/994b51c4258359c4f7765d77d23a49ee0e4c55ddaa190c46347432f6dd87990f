# Card 16 — innerrun-api-error-revive

**Edit:**
- `internal/battenshed#InnerRunDeps`
- `plan:internal/battenshed#innerRunMemory`
- `internal/battenshed#innerRunProducer.noticeStep`
- `internal/battenshed#innerRunProducer.judgeAgents`
- `internal/battenshed#TestNotice_APIErrorStall`
- `internal/battenshed/doc.go`
- `internal/testkit/envkit#nilLegal`

**Intent:**
Revive a session stalled on an API error before telling the orch (D9).
`InnerRunDeps` gains `ReviveAPIError func(ctx context.Context) error`, which sends tender's revive line to the task pair's stalled sessions through the conditioned send; a nil one sends nothing and keeps today's notice-at-first-sight, so `envkit`'s `nilLegal` seam inventory gains `InnerRun.ReviveAPIError` and `TestFullEnv_FillsEverySeam` keeps passing.
When `judgeAgents` finds the api-error condition for an episode (its key, the newest agent activity) the memory record has not revived, `noticeStep` calls `ReviveAPIError` once, records the episode as revived and sends no notice; a revive error is warned once per episode and still counts as the revive.
The notice goes when the same episode still stands at the next due probe check after its revive, or on the third revived episode in a row with no write of the child's status file between them: the memory record keeps the count of revived episodes in a row and the status file's modification time it last saw, and a status write resets the count.
Under a covering pause no revive is sent, since the running-child conditions are not judged then.
`doc.go`'s notice paragraph states the revive-first rule.
`TestNotice_APIErrorStall` gains rows over successive passes: the first sight revives and notices nothing, the episode still standing at the next probe notices once, an episode that ends after the revive notices nothing, and three revived episodes with no status write notice on the third; a status write between them resets the count.

**ImpactSummary:** One new optional seam and a revive step ahead of one notice condition; the other conditions and their once-per-episode markers are unchanged.
