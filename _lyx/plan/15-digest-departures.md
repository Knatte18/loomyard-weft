# Card 15 — digest-departures

**Create:**
- `plan:internal/websterengine#CommitLogEntry` -> `type CommitLogEntry struct`
- `plan:internal/websterengine#CommitLogEntry.DepartureLines` -> `func (e CommitLogEntry) DepartureLines() []string`
- `plan:internal/websterengine#realGit.CommitLog` -> `func (realGit) CommitLog(worktree, base, head string) ([]CommitLogEntry, error)`
- `plan:internal/websterengine#commitLog` -> `func commitLog(worktree, base, head string) ([]CommitLogEntry, error)`
- `plan:internal/websterengine#batchDepartures` -> `func batchDepartures(geom Geometry, startSHA, headSHA string) ([]string, error)`
- `plan:internal/websterengine_test#fakeGit.CommitLog` -> `func (g *fakeGit) CommitLog(_, base, head string) ([]websterengine.CommitLogEntry, error)`
- `plan:internal/websterengine_test#fakeGit.commitWith` -> `func (g *fakeGit) commitWith(subject string, departures ...string) string`

**Edit:**
- `internal/websterengine#Git`
- `internal/websterengine#Digest`
- `internal/websterengine#RecordBatch`
- `internal/websterengine#PersistRecoveryTerminal`
- `internal/websterengine#digestSummaryLine`
- `internal/websterengine#RenderProgress`
- `internal/websterengine/doc.go`
- `internal/websterengine_test#fakeGit`
- `internal/websterengine_test#fakeGit.commit`
- `internal/websterengine_test#newRecordFixtureOver`
- `internal/websterengine_test#newRecoverFixtureOver`
- `contracts/stencils/webster/webster-template-master.md`
- `contracts/stencils/webster/webster-body-implementer.md`
- `internal/websterengine#TestRepositoryProbes`
- `internal/websterengine_test#TestRecordBatch_ParentMovedHead`
- `internal/websterengine_test#TestRecoverBatch_TerminalRunsTheSamePostBatchChecksAsRecordBatch`
- `internal/websterengine_test#TestBeginBatch_PromptFile`
- `internal/websterengine_test#TestRenderBatchLists`
- `internal/websterengine_test#TestMasterTemplate_QuotesBulletLists`

**Uses:**
- `internal/gitrepo#Repo.Log`
- `internal/gitrepo#LogQuery`

**Intent:**
D2.3 and D2.4: a fork's `Pattern-Departure:` trailers reach the Master and every later fork through Go.

- `CommitLogEntry` holds `SHA`, `Subject` and `Departures`, the values of the commit's `Pattern-Departure:` trailer lines in order, each verbatim whatever its content; `DepartureLines` renders each as `<subject>: <value>`.
- `Git` gains `CommitLog(worktree, base, head)`, base and head full SHAs: the non-merge commits on head's first-parent line after base, oldest first; a base not on that line, or a SHA the repository does not hold, is an error.
  `realGit.CommitLog` calls `commitLog` in `gitwrap.go`, the one webster file that reads git, which reads through `internal/gitrepo` as the record's Constraints name: `gitrepo.New(worktree).Log(head, gitrepo.LogQuery{FirstParent: true})`, a read-only go-git walk, stopped at `base`, keeping each commit with fewer than two parents, and reversed to oldest first.
  Each kept commit's subject is its message's first line, and its departures are the values of the `Pattern-Departure:` lines of the message's last paragraph, where git reads trailers, each trimmed and otherwise verbatim.
  A walk that reaches the root without meeting `base` is an error naming both SHAs.
- `batchDepartures` returns every `DepartureLines` of `geom.git().CommitLog(geom.WorktreeRoot, startSHA, headSHA)`, nil for an empty `startSHA` (a record from before the field), so a commit before the batch's start head, or a parent merge-in's commits within the range, is never attributed to the batch.
- `Digest` gains `Departures []string` tagged `json:"departures,omitempty"`, after `Deviations`, informational and never changing a batch's status (`PATTERN-tolerant-records`: an older `state.json` without it loads).
- `RecordBatch` fills it from `bs.StartSHA` to the report's resolved head, beside `Deviations`; `PersistRecoveryTerminal` fills it from `bs.StartSHA`, which a recovery inherits from the stuck fork, to the recovered head, so the whole bracket's departures carry over.
  A `CommitLog` error is returned with "way forward: transient, re-run the verb", before the batch is marked terminal.
- `digestSummaryLine` appends ` pattern departures=<lines joined by "; ">` when the list is non-empty, so the next fork's prior-batch line carries them; `RenderProgress` appends ` (pattern departures: <lines joined by "; ">)` to a terminal batch's line whose digest holds any, so a resumed Master's trail carries them.
- `webster-template-master.md`'s quoted digest field list gains `departures` after `deviations`, a sentence says it lists the batch's `Pattern-Departure:` trailers, informational like `deviations`, and the summary rule names each batch's departures beside its deviations; `webster-body-implementer.md`'s prior-batch paragraph says the line's `pattern departures` are the preceding batch's trailers.
- `doc.go` describes the field, its range and its two readers.

Tests:
`fakeGit` gains a `logs` map from SHA to `CommitLogEntry`; `fakeGit.commit` registers an entry with an empty subject, `commitWith` registers one with a subject and departures, and `fakeGit.CommitLog` walks head's first-parent chain down to base, skipping commits with two parents.
`TestRepositoryProbes` gains a subtest over a real repository: two commits after a base, one with two `Pattern-Departure:` trailer lines, one whose trailer value is malformed text, and a merge of a side branch whose commit carries a trailer; `commitLog` returns the two first-parent non-merge commits oldest first with their trailer values verbatim and nothing from the side branch or the base, and a base that is not on head's first-parent line is an error.
`TestRecordBatch_ParentMovedHead` gains rows: a trailer on a commit before `StartSHA` and one on a parent merge-in's side commit are not recorded, a batch commit's trailers are, a batch with none records no `departures` key in the persisted state, a record with an empty `StartSHA` records none without calling `CommitLog`, and a `StartSHA` the fake does not hold returns the transient way forward with the batch left non-terminal.
`newRecordFixtureOver` and `newRecoverFixtureOver` keep their callers' `startSHA` on head's first-parent chain: where an existing caller hands a SHA the fake or the scratch repository does not reach from head, the helper registers it as head's ancestor, so `CommitLog` answers every existing row.
`TestRecoverBatch_TerminalRunsTheSamePostBatchChecksAsRecordBatch` asserts a recovered batch's digest carries the fork commit's and the recovery commit's trailers.
`TestBeginBatch_PromptFile` gains a row whose predecessor digest holds departures, and `TestRenderBatchLists` a progress row whose done batch holds them.
`TestMasterTemplate_QuotesBulletLists`' digest want adds `departures` after `deviations`.

**ImpactSummary:** one new `Git` method with one real and one fake implementation; `record-batch` and `recover-batch` each make one more read-only git call, and the digest gains one optional key every reader tolerates.
