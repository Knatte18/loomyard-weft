# Card 7 — decision-add-darn

**Create:**
- `plan:internal/loomcli#decisionWayDarnWriter` -> `const decisionWayDarnWriter = "way forward: <the text the Intent gives>"`
- `plan:internal/loomcli#decisionWayEntryLost` -> `const decisionWayEntryLost = "way forward: <the text the Intent gives>"`
- `plan:internal/loomcli#checkDarnRecord` -> `func checkDarnRecord(deps decisionDeps) (string, error)`

**Edit:**
- `internal/loomcli#decisionWayNoRecord`
- `internal/loomcli#decisionDeps`
- `internal/loomcli#decisionVerb`
- `internal/loomcli#decisionState.preRun`
- `internal/loomcli#loomCLI.decisionCmd`
- `internal/loomcli#decisionFake`
- `internal/loomcli#decisionFake.deps`
- `internal/loomcli#TestDecisionVerb_AppendsAndCommits`
- `internal/loomcli#TestDecisionVerb_Refusals`
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`

**Uses:**
- `plan:internal/loomcli#discussionHistory`
- `internal/discussionparser#CheckAddedDecisionsKept`
- `internal/discussionparser#ParseAddedDecisions`
- `internal/fabricengine#CommittedAnchoredFile`
- `internal/loomcli#addedDecisionHeading`
- `internal/loomcli#commitDiscussion`

**Intent:** Make `lyx loom decision add` work on a darn run once Darn-Write has committed its record, including while Darn-Write runs, bounded to a record the writer has validated and not touched since (D5).

`decisionDeps` gains four seams: `darn func() (bool, error)`, whether the target's run is a darn run; `readRecord func() ([]byte, error)`, the record's working copy; `committed func() ([]byte, bool, error)`, the record as committed at HEAD, found false when it is not committed; and `history func() ([][]byte, error)`, every committed version of the record.
`decisionState.preRun` fills them for the resolved target: `darn` reads `shedrun.ReadSeed(target, shedrun.SelfRunID)` and compares `resolveSeedRecipe(seed, found)` with `shedrun.RecipeDarn`; `readRecord` reads `recordPath`; `committed` is `fabricengine.CommittedAnchoredFile(target, loomengine.DiscussionDecisionRecordRel())`; `history` is `discussionHistory(target)`.

`decisionVerb`'s caller checks, `--by` checks and the Discussion-Write running refusal stay first and unchanged, so a task session other than the parent or arbiter is still refused on a darn run.
Then, when `deps.darn` reports a darn run, `checkDarnRecord` returns a refusal text or an error before any append:
a working copy that does not exist is the no-record refusal;
a record not committed (`committed` found false), a working copy whose bytes differ from the committed copy, and a working copy for which `CheckAddedDecisionsKept(working, history)` reports findings (naming each dropped heading) are each refused with `decisionWayDarnWriter`.
A `darn`, `readRecord`, `committed` or `history` error is refused with the existing `decisionWayRetry`.
After the append and the commit, a darn run reads `committed` again and parses it with `ParseAddedDecisions`; when no entry carries the heading `addedDecisionHeading` gives for the appended entry, the verb refuses naming that heading with `decisionWayEntryLost`, which covers a writer write landing between the append and the commit; a read or parse failure there is refused with the same way forward, since the entry may already be committed.
A loom run skips both darn steps and behaves exactly as today.

The way forwards, each a trailing `way forward:` clause:
`decisionWayNoRecord` drops "a darn run never has one, so name the call in your report to the run's parent instead" and adds that on a darn run the record exists once Darn-Write has validated it, and until then the caller messages the running Darn-Write session or, with no writer live, records the call as a board note naming the task;
`decisionWayDarnWriter` says Darn-Write has not finished its record or is editing it after validating it, to message the running Darn-Write session, and to re-run the verb once the committed record is unchanged;
`decisionWayEntryLost` says a Darn-Write write landed during the append, to message the running Darn-Write session to leave the record alone, and to re-run the verb.
None names a command the agents' settings deny.

`decision.go`'s file comment and `decisionCmd`'s `add` `Long` state the darn behavior and its bound: the caller rule and the append-only, labelled, committed entry are unchanged; the one new edge is appending while Darn-Write runs, bounded by the committed-copy and history checks to a record the writer validated and has not touched, and by the post-commit check to an entry that reached the weft.

`decisionFake` gains a darn flag and error, a `readRecordErr`, a committed copy with its found flag and error, a separate error for the committed read after the commit, history versions and their error, a count of darn seam calls, and a flag that makes the next commit leave the committed copy without the appended entry, and a field holding the bytes the commit leaves as the committed copy; its `deps()` wires the four seams, `readRecord` returning `readRecordErr` when set and the fake's record file otherwise, and the fake's commit setting the committed copy to that field's bytes when it is set, leaving it unchanged when the flag is set, and copying the file's bytes otherwise.
`TestDecisionVerb_AppendsAndCommits` gains darn rows: from an unnamed shell and from the recorded parent over a committed, unchanged record, and one with the status running at `Darn-Write`, each appending and committing once.
`TestDecisionVerb_Refusals` gains a `notWant` list and an `afterCommit` flag, which drops the untouched-record and zero-commit assertions for the rows that refuse after their commit, and these rows: the loom "no decision record" row wants "Darn-Write has validated it" and not "never has one"; a darn record missing (`readRecordErr` wraps `fs.ErrNotExist`), wanting the darn clause of the no-record way forward; a darn record not committed; a darn working copy differing from its committed copy; a darn record missing an entry a history version holds, naming the heading; one row each for a `darn`, `readRecord`, `committed` and `history` error, each refused with `decisionWayRetry` and no append; the post-commit row, refused naming the heading, with `afterCommit` set; a post-commit committed copy, set through the fake's committed-bytes field, holding an `Added after Discussion` heading with no date, which `ParseAddedDecisions` fails on, refused with `decisionWayEntryLost`, with `afterCommit` set; and a post-commit committed-read error, refused with `decisionWayEntryLost`, with `afterCommit` set.
The existing "named caller differing from the recorded parent" row sets the darn flag and asserts the fake's darn seams are never called, pinning that a task session is still refused on a darn run before any darn read.

`contracts/specs/refusal-spec.md`'s loom section rewrites the `decision: no decision record` row's way forward to the new clause and adds rows for the darn record unfinished (not committed, differing from its committed copy, or missing a history entry), the entry missing from the committed record after the commit or that record unreadable, and a transient row for a failed read of the run's recipe, the record, its committed copy or its history.
`docs/overview.md`'s `decision add` line says the verb also works on a darn run once Darn-Write has committed its record, under those checks.

**ImpactSummary:** `decision add` gains darn-only checks before the append and after the commit and a rewritten no-record way forward; a loom run's path, the caller rule and the success envelope are unchanged.

**Commit:** `7: decision add works on a darn run once its record is committed`
