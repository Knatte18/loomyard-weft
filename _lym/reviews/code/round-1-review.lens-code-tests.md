```yaml
lens: code-tests
outcome: reported
```

Task: review the diff's tests under the code-tests lens. For each behaviour the diff adds or changes, I named the test row that pins it, or none.

**Behaviour → pinning test**

1. `gitrepo.Fetch` prunes the tracking ref of a branch deleted on the remote and keeps the live branch's ref. Pinned by `TestFetch_BranchDeletedOnRemote_DropsItsTrackingRef` (`internal/gitrepo/fetch_integration_test.go:74-106`).
2. `gitrepo.Fetch`'s error now tells the user to run `git -C … fetch --prune` (`internal/gitrepo/pull.go:49`). No test pins it: a grep of `fetch_integration_test.go` finds no assertion on that text. This is diagnostic text only, so I raise no finding.
3. `judgeAfterCodeTipFetch`, case by case:
   - A code-tip wait fetches, then the fresh read acts: row `AFetchThatDropsAStaleRecordsRefLetsTheRunAct`.
   - The tip is still missing after the fetch, so the run waits: row `ATipStillMissingAfterTheFetchWaits`.
   - A failed fetch keeps the first verdict and does no second read: row `AFailedFetchKeepsTheCodeTipWait`.
   - Any other wait fetches nothing: row `AnotherWaitFetchesNothing`.
   - An act verdict fetches nothing: row `AnActVerdictFetchesNothing`.
   - A failed first read returns its error: row `AFailedReadReturnsItsError`.
   - A failed second read, after a successful fetch, should return its error. **No row** (finding 2).
4. The wiring in `hubRunVerdict`: `read` is `pairteardown.ReadRun`, `fetch` is `fetchRunForCatchUp`, and the fetch reaches the records repository so a stale `origin/<slug>-weft` is pruned before the second judgement. **No test** (finding 1). The reach report lists only `TestArmAt_RunOwnedElsewhereWritesNoSeed` for `hubRunVerdict`, which does not reach the code-tip branch. Every `judgeAfterCodeTipFetch` row uses fake `read` and `fetch` (`internal/battencli/arm_runverdict_test.go:62-72`).
5. `runReader.read` sets `CodeTipPairAbsent = !pairHere` (`internal/pairteardown/runread.go:249`):
   - True with no pair: row `AMissingCodeTipSetsTheThreeCodeTipInputs`.
   - False with a pair here: row `AMissingCodeTipWithAPairHereLeavesThePairAbsentInputClear`.
   Both values are pinned.
6. `Act` copies `CodeTipPairAbsent` from `Inputs` onto `Verdict` (`internal/runfsm/act.go:116`). Pinned indirectly by `TestAct` row `MissingCodeTipWithNoPairHereNamesTheRecordsFetchFirst`: its refusal text depends on the flag.
7. `codeTipRefusal`, case by case:
   - Pair absent, machine named: row `MissingCodeTipWithNoPairHereNamesTheRecordsFetchFirst`, exact text (`internal/runfsm/act_test.go:39-42,70`).
   - Pair here keeps the old wording: rows `MissingCodeTipWaits` and `MissingCodeTipWithoutMachineLeavesOutThePushClause`.
   - Pair absent, no machine named: **no row** (finding 3). It yields a distinct way forward ending `…; if the refusal stays, run <retry> again once the code push lands`.
   - Empty `CodeTip` gives the reason without a sha (`internal/runfsm/act.go:202`, the mutation survivor): **no row**. This is pre-existing behaviour moved into a new function, not behaviour the diff adds, so it is no finding under the rubric's mutation rule.

**Findings**

1. MEDIUM, `design`, `internal/battencli/arm.go:634-638` (`hubRunVerdict`). The behaviour the board entry asks for has no test that would fail if it broke: a removed slug's stale records ref is pruned by the act check's fetch, and the run then acts. All `TestJudgeAfterCodeTipFetch` rows fake both `read` and `fetch` (`internal/battencli/arm_runverdict_test.go:62-72`). The reach report lists no test of `hubRunVerdict` that reaches the code-tip branch. Nothing asserts that `fetchRunForCatchUp` prunes the ref `OriginCodeTip` reads in the case with no pair, which goes through `gitrepo.New(fabricengine.RecordsWorktree(prime)).Fetch()` (`internal/battencli/catchup.go:224`). Replacing the `fetch` closure with a no-op, or with a fetch of the wrong repository, passes every test. The fix is an integration row:
   - a hub whose records store holds `origin/<slug>-weft` with a transition naming a code tip origin lacks;
   - the branch is deleted on origin;
   - assert `hubRunVerdict(prime, slug)()` returns `AnswerAct`.
2. LOW, `design`, `internal/battencli/arm.go:662` (`return judge()` after a successful fetch). No row has the second read fail. The rows that read twice (`AFetchThatDropsAStaleRecordsRefLetsTheRunAct` and `ATipStillMissingAfterTheFetchWaits`) have `readErr` nil. `readErr` fails every read, the first included (`internal/battencli/arm_runverdict_test.go:62-66`). The fix is a row where the first read gives the code-tip wait, the second read fails, and the test asserts the error with `wantFetch` 1 and `wantReads` 2.
3. LOW, `design`, `internal/runfsm/act.go:211-218` (`codeTipRefusal`). The pair-absent wording with no `CodeTipMachine` has no `TestAct` row. In that case the way forward joins the fetch step to the bare `run <retry> again once the code push lands`, and no row pins that text (`internal/runfsm/act_test.go:68-70`). The fix is a row with `CodeTipPairAbsent: true` and `CodeTipMachine` empty, asserting the exact refusal.

**Tool calls**

1. A grep of `internal/gitrepo/fetch_integration_test.go` for assertions on the fetch error's reproduction text, and of the battencli tests for other tests of `judgeAfterCodeTipFetch` or `hubRunVerdict`. My context held the diff hunks and the reach report, but not the test file's assertions or a list of every caller in the test files.
