```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked each claim, guard and bound in the decision record and support log.

**Findings**

1. **BLOCKING — D7, which also breaks D8 and D9.** The guard catches more Bouncers than the Discussion one.
   - **The rule:** D7 requires `approve_seam: discussion` on every Bouncer row whose `artifact_paths` name `_lym/discussion/decision-record.md`.
   - **What it catches besides Discussion-Bouncer:**
     - tapestry's Code-Bouncer: `contracts/recipes/tapestry-recipe.yaml`, artifact_paths at lines 468–469 list the record and `_lym/plan`.
     - darn's Code-Bouncer: `contracts/recipes/darn-recipe.yaml`, lines 143–144.
     - sampler's Code-Bouncer: `contracts/recipes/sampler-recipe.yaml`, lines 82–83.
   - **Consequences of following the rule:** darn and sampler runs would store their body-copy record and flag the entry `discussed` at Code-Review approval, which contradicts Scope Out and D8 ("a darn run, a sampler run … are never copied"). D8 and D9 would also no longer have one approving row: by the rows rule, darn would count as "past approval".
   - **Consequences of not following it:** the guard fails on today's recipes.
   - **Fix:** key the guard on something that names Discussion only, such as the segment holding a DiscussionWrite row's review, or the Bouncer of the segment entered from a DiscussionWrite row. Use the same rule to find "the approving row" in D8 and D9.

2. **MEDIUM — D9.** "Admits the callers `decision add` admits, through the same caller check" cannot work as written.
   - `decision add`'s check depends on `--by`: the arbiter strand passes only with `--by arbiter`, and the sampler strand only with `--by operator`.
   - D9 defines no `--by`, so the plan writer must invent the arbiter and sampler handling.
   - Fix: state the caller set outright, for example the parent orch, the operator's unnamed shell and the run's own arbiter strand. The sampler is moot because the sampler recipe has no Discussion approval.

3. **LOW — D4/D5, a state an older binary leaves behind.**
   - An older binary on another machine that prunes or removes a discussed entry drops the entry but not `decision-&lt;slug&gt;.md`.
   - The orphan file then stays on origin indefinitely. D5's "lost when origin lacks the entry" check runs only on commits that carry the file.
   - It is harmless, since `DecisionRecord` requires the flag, but D1's claim that the file follows its entry holds only for new binaries. Worth one line in the bounds.

4. **LOW — D4, merge with the same slug.**
   - The record does not say whether a merge counts its removed slug by net change or per `RemoveTask` call.
   - `MergeTasks` may remove slug X and upsert X again. The upsert goes through `NewTask`, so the new X has `discussed` false.
   - Judged per `RemoveTask` call, the file is deleted. Judged by net change, the file stays without a flag.
   - Fix: say "judged by net change, as the run lock is".

5. **LOW — D10, wrong message.** An entry flagged `discussed` whose file is missing reads "not discussed" through `DecisionRecord`, so `--record`'s refusal names the wrong cause. Name "flagged, record file missing" separately, or accept it as within D2's invariant.

**Claims attacked**

- Every approving Bouncer path calls the approve seam: held. All four call sites go through `approveAndCommit`.
- The seam runs after carry-over and before the commit: held. `settle` (converged) and `acceptRound` call `writeCarryOver`, then `approveAndCommit`, then `finishSettle`. `approveWithoutReview` has no carry-over, which is consistent.
- A failed store halts the run and a resume re-stores: held. `finishSettle` comes after approve, so a re-call settles again, as with `ApprovePlan`.
- Board-lock contention from the run process: held. `flock.AcquireWriteLock` blocks rather than failing fast, so a running detached sync does not halt the run.
- "Never a flag without a file": held. Sync takes `board.lock`, so it cannot commit between the file write and the save. The D4 `afterSave` removal is also under the lock.
- Replay of a record-only commit, where the flag is already true and `board.json` is untouched: held. `parseReplayBoard` treats empty bytes as no entries, so no entry change or removal is replayed.
- Render never deletes the record file: held. `RenderToDisk` deletes only names in the previous manifest.
- The run lock lets the flag through: held. `scopeChanged` lists fields explicitly, and `discussed` would not be among them.
- The D8 race bound: held as stated. The only uncovered window is between the approve read and the driver moving the current row. A `decision add` earlier in the review is picked up by the approval store.
- D8 never copies beam project records: held. pawl has no Discussion Bouncer.
- D7 and D8 pick one approving row: broken (finding 1).
- D9's caller rule: broken (finding 2).
- D1's "the file follows its entry": broken for older binaries (finding 3).

**Tool calls**

- Grepped `artifact_paths` across `contracts/recipes/*.yaml` and read `AcquireWriteLock` in `internal/lock/lock.go`. My context had only the Discussion-Bouncer and Plan-Bouncer artifact paths, and not the lock's blocking behaviour.
- Read the converged settle and `acceptRound` in `internal/shedadapters/bouncer.go`, `parseReplayBoard` in `replay.go`, and tried to grep `stampDates`, which matched nothing. My context had no ordering evidence for carry-over against approve on the accept path, and no evidence for how replay handles an empty `board.json`.
