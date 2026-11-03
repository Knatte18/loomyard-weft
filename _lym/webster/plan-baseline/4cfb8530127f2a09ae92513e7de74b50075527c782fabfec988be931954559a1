# Card 8 — decision-add-copies

**Create:**
- `plan:internal/loomcli#decisionWayStore` -> `const decisionWayStore = "way forward: once the cause above is cleared, run \"lym loom decision store\" to copy the committed record onto the board entry"`
- `plan:internal/loomcli#copyRecordToBoard` -> `func copyRecordToBoard(deps decisionDeps, currentRow string) string`
- `plan:internal/loomcli#TestDecisionVerb_CopiesRecordToBoard` -> `func TestDecisionVerb_CopiesRecordToBoard(t *testing.T)`

**Edit:**
- `internal/loomcli#decisionDeps`
- `internal/loomcli#decisionVerb`
- `internal/loomcli#amendmentCall.afterFailedCommit` -> `func (a *amendmentCall) afterFailedCommit(deps decisionDeps, added []discussionparser.AddedDecision, commitErr error, wayCommit string) (landed bool, refusal string)`
- `internal/loomcli#decisionState.preRun`
- `internal/loomcli#decisionFake`
- `internal/loomcli#decisionFake.deps`
- `internal/loomcli#loomCLI.decisionCmd`
- `internal/loomcli#TestDecisionVerb_CommitFailureNamesWayForward`
- `internal/loomcli#TestDecisionVerb_AmendsPlan`

**Extends:**
- `internal/loomcli/doc.go`

**Uses:**
- `plan:internal/loomcli#storeDecisionRecord`
- `internal/loomcli#newDecisionFake`
- `internal/loomrecipe#PastDiscussionApproval`
- `internal/boardengine#ErrRecordEntryDone`

**Intent:**
Make `lym loom decision add` refresh the board copy of the record (D8).

**New deps.**
`decisionDeps` gains two fields:
- `pastApproval func(currentRow string) bool`, which reports whether the run has passed its Discussion approval as `loomrecipe.PastDiscussionApproval` judges it.
- `storeRecord func(record []byte) error`, which copies bytes onto the run's board entry.
Both are always filled: `preRun` fills them for every call, and `decisionFake.deps` fills them for every test, so no nil meaning is defined.

`decisionState.preRun` fills them.
- `pastApproval` calls `PastDiscussionApproval(recipe, currentRow)`, returns `past` and drops the error, with one comment naming the reason: `resolveReviewTarget`'s `recipeUnknown` refusal, which `preRun` runs before it builds any dep, refuses every recipe the accessor cannot parse. A recipe with no approval row (darn, sampler) answers false, and so does pawl, whose `runRecordOf` gives the beam's project record. So none of those runs is copied; card 6's rows pin that rule over the recipes' rows.
- `storeRecord` calls `storeDecisionRecord(target, seedSlug(target.WorktreeName), record)`.

**The copy.**
`copyRecordToBoard(deps, currentRow)` asks `pastApproval(currentRow)`.
- False: it returns "".
- True: it reads `deps.committed()` and passes the whole committed record to `storeRecord`.
A not-found answer from `committed()` likewise has no text of its own and is reported as a read error: the copy runs only after `deps.commit()` succeeded or `afterFailedCommit` found every entry at HEAD, so HEAD holds the record.
It never fails the verb, since the entry is already committed. It returns the text the success envelope reports as `record_store_error`, and logs each failure at Warn:
- an error matching `boardengine.ErrRecordEntryDone` gives `the entry has landed and keeps the record it last stored`, with no pointer at the store verb;
- any other error, from the read or the store, gives the error followed by `; ` and `decisionWayStore`, whose "once the cause above is cleared" defers to a way forward the error itself names, such as the board's restore-the-entry step.

**Where it runs.**
`decisionVerb` calls it after the existing post-commit `committedLacksEntry` check and before `notifyLiveRuns`, with `st.CurrentProducer` from the status it read (empty when no status file).
It sets `result["record_store_error"]` when the text is non-empty.
The path where `afterFailedCommit` finds the commit landed falls through to the same call, so it copies as a successful commit does.

On a run past its approval, every failed-commit refusal that sends the caller to `lym fabric commit` also names the store: `decisionVerb` computes the way once, `decisionWayFabricCommit` followed by `, then "lym loom decision store" once "lym fabric commit" has landed the record` when `pastApproval(currentRow)` is true, and `decisionWayFabricCommit` alone otherwise.
It uses that way in the no-amendment refusal and passes it to `afterFailedCommit`, whose three refusals that today end in `decisionWayFabricCommit` end in `wayCommit` instead; `decisionVerb` is its one caller.
`rollBackAppend` keeps `decisionWayFabricCommit`: its commit lands only the restored amendment record, never an entry to copy.

**Docs.**
`decision.go`'s file header gains a paragraph on the copy and its bound: a call landing while Discussion-Bouncer is settling is not copied, and the store verb closes that window by hand.
It also says a run a `lym loom goto` moved past Discussion-Bouncer counts as past, so its record is copied unapproved, as the parent's goto decided.
`add`'s `Long` help names `record_store_error`.
`loomcli/doc.go` gains one sentence.

**Tests.**
`decisionFake` gains `pastRows map[string]bool`, `storeErr error` and the recorded `askedRows []string` and `stored [][]byte`, filled into `deps()`: its `pastApproval` records the row it is asked about and answers `pastRows[row]`.
Rows set the run's row through the fake's existing `status.CurrentProducer`, so a row that passed the wrong row to `pastApproval` fails.

`TestDecisionVerb_CopiesRecordToBoard` (tier 0) is a table over `newDecisionFake` with the committed copy seeded:
- At a row in `pastRows`, the stored bytes equal the committed record holding the new entry, `askedRows` holds that row, and the envelope has no `record_store_error`.
- At `Discussion-Burler-Fix`, a row not in `pastRows`, nothing is stored.
- A failing store exits 0 with `record_store_error` naming `lym loom decision store`.
- A store failing with `ErrRecordEntryDone` exits 0 with `record_store_error` that does not name `lym loom decision store`.

`TestDecisionVerb_CommitFailureNamesWayForward` gains rows: a failed commit at a past row is refused naming `lym fabric commit` and then `lym loom decision store`, and at a row not past names no store verb.

`TestDecisionVerb_AmendsPlan` gains rows on its own setup: an `--amends-plan` call whose failed commit `afterFailedCommit` finds landed stores the committed record; and one whose status commit landed the amendment record without the entry, at a past row, is refused naming `lym loom decision store` after `lym fabric commit`.

**ImpactSummary:** `decision add` gains one board write after its commit, one envelope key, `record_store_error`, and the store pointer on its failed-commit refusals past the approval, all inside loomcli; `afterFailedCommit`'s one caller is `decisionVerb`. Darn, sampler and beam records are never copied. A copy failure never changes the verb's exit code, so its callers in the orch role and the stencils see the same success shape.
