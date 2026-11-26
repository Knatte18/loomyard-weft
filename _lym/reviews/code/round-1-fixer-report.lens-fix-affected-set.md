```yaml
lens: fix-affected-set
outcome: reported
```

Task: checked, through the affected-set lens, whether the fix covers every member of each finding's set.

**Unchecked findings:** F3, F5, F6 and F10 had no search rerun. F6's four branches and F10's blast radius I judged from context only. F4 is a single site, not a set.

**Findings**

1. **MEDIUM.** Changed text: `waiting.go` `resumeStep` and the `--answer` requirement. The Master template's re-entry rung for a `waiting` batch was left as it was.
   - `contracts/stencils/webster/webster-template-master.md`, the `waiting` bullet under "Read the trail by status" (about line 80), still says "treat the step as taken: resume the batch with the verb named after `resume:`".
   - For a recovery batch that verb is now `recover-batch N --answer "<the parent's answer>"`, and a re-entered Master holds no answer.
   - Run without `--answer`, it is refused. Run as printed, the literal placeholder is recorded as the parent's answer and rendered into the recovery prompt.
   - The rung's "A strand that still finds the step untaken reports it again" assumes the old uncontested resume.

2. **LOW.** Changed text: the resume refusal in `RecoverSpawnOrAttach`. Several way-forwards still name a bare `recover-batch N`, which is now refused on a waiting recovery.
   - The transient messages in `internal/websterengine/recoverbatch.go` at about lines 273, 278, 283, 325, 359 and 596 can all fire while resuming one.
   - So can the reed-boot message at `internal/webstercli/recoverbatch.go:228`.
   - `existingReportRemedy` (`internal/websterengine/beginbatch.go`, the `!recorded.Terminal` recovery arm) returns the bare verb instead of `resumeStep`.
   - The follow-up refusal does name `--answer`, so the operator is not stuck.

3. **LOW.** Changed text: the `AmendedCard` type doc in `internal/runplan/state.go` (F8).
   - It says "one card of a begun, not-done batch that a rebaseline accepted an edit to".
   - After F7, a terminal failed or stuck batch is begun and not done, yet records no amendment.
   - The field doc ("in flight or terminal dead") is the accurate one.

4. **LOW.** Changed text: the new `BatchState.ParentAnswers` field.
   - The record rebuild in `BeginBatch` (`internal/websterengine/beginbatch.go`, about lines 404-428) carries `Recoveries`, `RecoverySessions`, `RecoveryEngines` and `RecoveryStartSHA`, but not `ParentAnswers`.
   - A re-begin therefore drops the parent's earlier answers before the next recovery.

I found no gap in F7's code or doc sites, or in F9's overview-edit sites. The `recordbatch.go` comments "A failed batch records no amendment" are true again.

**Tool calls**
1. Grep for `recover-batch N` way-forwards: the context showed only some of the sites.
2. Grep for amendment and re-arm wording across code, contracts and docs: other readers and docs were not in context.
3. Grep for `NEEDS_PARENT`, waiting and asking sites: the post-fix set was not in context.
4. Grep and `sed` for `BatchState` construction sites: the `BeginBatch` rebuild was not in context.
5. Grep for overview-frame wording: the post-fix F9 sites were not all in context.
6. Read the Master template's resume section, the `rebaseline` help and `existingReportRemedy`: none of them was in context.
