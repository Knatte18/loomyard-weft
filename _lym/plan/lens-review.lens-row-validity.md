```yaml
lens: row-validity
outcome: reported
```

Lens: row-validity. I checked whether each planned test row can reach the branch it claims, whether its assertion would fail on wrong code, and whether it repeats a row an existing test already has.

**Findings**

- **MEDIUM — Card 6 (D12, Finalize size): the operand is wrong, and the row cannot tell the right one from the wrong one.**
  - The card says `RecordLandedSize` gets "the parent tip … the value `readParentMergeBefore` gives". That function reads the task head, the task's merge base and the earlier merge record, never the parent tip (`internal/landingshed/finalize.go:1074-1089`).
  - The test fake `recordingParentMerger` scripts a single `headSHA` (`internal/landingshed/finalize_test.go:36-38`). It can produce no separate pre-merge parent tip, so the row "gets the pre-merge parent tip and the landing commit" has no distinct value to compare against.
  - The card should name the operand: either the task's merge base and task head, or a parent HEAD read before `mergeParent`. If it is the latter, the card should list the fake field that scripts it.

- **LOW — Card 6: the failure row belongs in a different test.**
  - "A row where it fails and the verdict stays Done" is planned in `TestFinalize_RecordsTheLandingCommitOnTheFirstTry`.
  - The failure-keeps-Done behaviour already has its table in `TestFinalize_RecordLandingFailureKeepsDone` (`finalize_test.go:510-533`). The row should be a third entry there.

- **MEDIUM — Card 3: the stop path cannot fill the `SessionEnd` fields the card prescribes, and its row asserts only the outcome.**
  - `RunState` carries `Engine`, `RoleKey` and `CreatedAt`, but no model and no role segment (`internal/shuttleengine/rundir.go:100-166`).
  - `stopRecorded` has no `Spec` to read them from (`internal/shuttleengine/stop.go:76-90`).
  - The card still says the hook is called "with the usage the record's engine's `UsageReader` reads" and the same `SessionEnd` fields, which includes Role and Model.
  - The stop row needs three things the card does not list:
    - a run record on disk naming the `usageEngine`'s engine name;
    - a runner whose `EngineSet` resolves that name;
    - a statement of which fields are empty on this path.
  - Without those, the row passes whether or not Role and Model are carried.

- **LOW — Card 3, test economy.** The rows "a done run, a stuck run and a died run each call the hook once" run `finalize` over the same fixture as `TestRun_Finalize_ReportsUsageAndTimes` (`internal/shuttleengine/wait_test.go:1595-1656`). They fit as hook assertions on that table's rows. Only the stop rows and the "no hook set" row need a new test.

- **LOW — Card 5, repeat.** `TestUsageRecorder`'s row "an engine with no readers entry writes counts from `SessionUsage` with an unknown cost" repeats card 4's `TestRecordSessionUsage` row "no transcript reader". The only new behaviour in `UsageRecorder` is picking `readers[end.Engine]` and the row, so that row should assert only the nil lookup being handed over.

- **MEDIUM — Card 8 (D13 hours): the hours of motif and raddle cells can never be reached.**
  - `ComputeTiming` selects runs only from `runRecipes`, which is tapestry and darn (`internal/reportengine/runs.go:23-24, 256`), and it requires a Finalize entry.
  - So every motif and raddle run gets unknown hours under card 8's join "by tag". A cell of such runs then reports no mean hours at all, but D13 says "in dollars and in active hours".
  - The row "a cell of runs with no landed record (means, no fit)" does not say whose hours it asserts. Over a `world` fixture of record-only runs, its hours mean cannot be produced.

- **MEDIUM — Card 14 and card 13: order and input for the row "a promotion of a note under a held slug (refused naming `lym board rename`)".**
  - The input D10 names, "a filed part made a note again", is a discussed note with a stored record, because filed parts are discussed (card 16).
  - Card 13's `checkDiscussedPromotion` refuses a plain promote of such a note first, with the `--recipe raddle` way forward.
  - Card 13 places it "beside `checkNewSlugs`" with no order. So the row either never reaches the store refusal or reaches it depending on check order.
  - The row needs to state its input (an undiscussed note, or `--recipe raddle`) and the cards need to fix the order of the two checks.

- **MEDIUM — Card 24: the refusal logic itself goes untested.** `TestStartGuard_Refuse` drives hand-built `startGuard` values whose seams are plain funcs (`internal/battencli/startguard.go:20-46`). A row that sets `Raddle` to return an error tests only the plumbing in `refuse`. The card does not say the rows build `Raddle` from `raddleStartRefusal` over a fixture entry and store. So the "no stored record → names motif", the "another created stamp / filed slug → names rename" and the own-stamp pass branches are never reached by any planned row.

- **LOW — Card 13: `TestPromote` cannot call the planned helper.** `TestPromote` is in the external package `boardengine_test` (`internal/boardengine/store_test.go:5,1869`). Its planned row calls the unexported `Store.promoteTo`, and the card lists no `export_test.go` entry. The row should go through `Board.PromoteTo` or `Store.Promote`, or the card should list the export.

- **LOW — Cards 13, 14 and 17: three test glyphs do not resolve.** `internal/boardengine#TestPromote`, `internal/boardengine#TestSlugRule` and `internal/boardcli#TestCLI` come back from quarry as not found, because those tests live in the external `_test` packages. The spellings should be `internal/boardengine_test#…` and `internal/boardcli_test#TestCLI`. The gate's `glyph-not-found` will catch these; I note them here only for row placement.

- **LOW — Card 14: an unlisted fixture helper.** `TestSlugRule` drives real `Board` writes in `boardengine_test` (`slugrule_test.go:33-60`). Its store-hit rows need a store directory with `store.json` written into the board path before card 16's `FileRaddle` exists. The card lists no helper or fixture for writing it, nor a board fixture with a nil seam for the "nil seam with a store hit" row.

- **LOW — Card 16, repeat.** `TestFileRaddle`'s row "a run-agent strand name still refused by `checkTaskCreation` on its own upsert" exercises an existing guard over an existing write path, already covered by `checkTaskCreation`'s own tests. `FileRaddle` changes nothing there.

- **LOW — Card 25: unlisted fake changes.**
  - The row "rerun after a mail with no marker (a second identical mail)" needs the `world` fake to stop or fail between the mail and the marker write. The card adds only "the hand-off seam and marker path" to `world`, with no marker-write error field.
  - The new required seam also has no refusal row in `TestNewProducers_RefuseAnUnsetSeam` (`internal/pawlengine/beamup_test.go:168`), which the card does not list.

**Tool calls**
1. `wait_test.go:1575-1660` and `stop.go:62-106`: to see the `usageEngine` fake, the finalize test's rows and `stopRecorded`'s body, for cards 1 and 3. My context had only summaries of them.
2. `rundir.go:100-166` and `finalize_test.go:493-533`: to check which fields `RunState` carries for the stop path, and the Finalize landing tests and their fake, for card 6.
3. `finalize.go:1074-1089` and the `recordingParentMerger` fake: to check what `readParentMergeBefore` returns and what the fake can script. My context had neither.
4. `lym quarry resolve` over the test and member glyphs the cards name: to confirm each named test or helper exists.
5. Grep of the test function names in the board, boardcli, pawl and loomshed test files: to locate the tests that did not resolve and their packages.
6. `slugrule_test.go:33-60` and the package clauses of the board tests: to see how `TestSlugRule` drives writes and which package each test is in.
