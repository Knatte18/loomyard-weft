```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: checking whether every sibling decision, criterion and cross-reference still agrees with this round's change set.

**1. [MEDIUM] D2's closed set picks up only half of `refuseAdoptLocalCheckedOut`**
- Changed text: D2's new closed-set bullet, "the `AdoptLocal` refusal of a task branch checked out in another worktree (`refuseAdoptLocalCheckedOut` …)", read with D1 step 3's new sentence, "The `AdoptLocal` refusal of a records branch checked out in another records worktree does not count that reused worktree as another."
- Summary: the same function also refuses a records branch checked out in a records worktree that is not the pair's. That twin refusal stays outside the closed set, so it falls into the transient bucket and fails every pass. F2 names the function, not just its warp half.
- Evidence: `internal/fabricengine/addadopt.go` returns the warp refusal ("branch %q is checked out in %s") and the records refusal ("records branch %q is checked out in the records worktree %s"). D2 lists only the first, and D2's "any refusal outside the closed set" is transient. No way-forward row in the "At any row" list covers the records case either.

**2. [LOW] D8 Bound still speaks only of the deleted ref**
- Changed text: D8 now "deletes that branch's remote-tracking ref … and unsets the branch's upstream config … in one step". Its Bound was left as it was: "the deleted ref names a branch origin does not hold, so no judge loses an origin head; a rollback does not restore it."
- Summary: the Bound does not say whether `Add`'s rollback restores the upstream config. Its last line, "`Add`'s step-11 replace … already drops the ref", now covers only half of D8's step.
- Evidence: D8's first paragraph against its Bound paragraph.

**3. [LOW] Scope bullets lag D7 and D8**
- Changed text: D7's new bullet ("Every push of the pair's weft branch from this machine outside a removal, the synchronous `commit-records` push included, takes the slug's teardown lock shared") and D8's extension to adopted branches and upstream config.
- Summary: two Scope bullets were not updated. One still lists only "`Add`, the task-branch discard and the cleanup sweep" as new takers of the teardown lock. The other still describes D8 as "The remote-tracking ref a former pair leaves under a branch name that `fabricengine.Topology.Add` forks anew". It covers neither an adopted branch nor the upstream config.
- Evidence: the Scope section of `decision-record.md` against D7 and D8.

**4. [LOW] Support log's question ledger still names a bare cancel**
- Changed text: D2, D3 and their criteria now name `lym batten cancel &lt;slug&gt; --remove`.
- Summary: Q5 in the support log's question ledger still reads "Name `lym batten cancel` in the way forward and in fabric remove's warning before Land", without `--remove`. The ledger is a record of picks, but a reader comparing it with D3 sees a different command.
- Evidence: the `support-log.md` question ledger, Q5.

Tool calls: none; my context already held everything this lens needed.
