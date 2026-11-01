```yaml
lens: way-forward
outcome: reported
```

Reviewed the plan's refusals, stops and recovery steps (way-forward lens). Five findings:

1. **MEDIUM — D8 / Card 5 `Remove` against `Settle`: removing a released front task can still run its action.**
   - `Remove` drops the record in step 2 and removes the board entry in step 3.
   - In between, a live dobby's `Settle` sees an open front entry with no record and records it again from the entry's own `on_land`. On the next tick the entry is gone, so it runs the action.
   - The worst case is a released, never-started fix task: `HasRun` is false, so the cancel step is skipped and the entry is open while both steps run. Card 12 step 5 removes exactly this case, and card 12 step 2 brought the dobby up just before.
   - Result: a green check's removal spawns one more `lym adopt check <subject>`. D8 says only a hand removal runs the action, and this one went through `fronttask`.
   - Neither `TestRemove` nor `TestSettle` covers the interleaving.
   - Evidence: card 5 Intent (Remove steps 1–3; the Settle bullet "records, from the entry's own `OnLand`, each open front entry of the project with no record"); card 12 steps 2 and 5.
   - Possible fix: run steps 2 and 3 under the record file's lock, which `Settle` also takes, or have `Settle` heal no record while a removal holds the lock.

2. **MEDIUM — D6 / Card 13 `failedEnd` case 5: a refused filing names no next step.**
   - "A filing error files nothing more and names it" gives the error but no way forward.
   - The holds stand, the check record reads failed and no fix task exists. Nothing starts by itself, so the operator needs to be told to fix the cause and re-run `lym adopt <subject>`, which resets the count and checks again.
   - Card 5's `File` refusal for an already-open front task also names no way forward ("naming that task's slug and that one front task stands open per worktree"). Card 13 case 3 normally catches that state first, but a filing that races a second check can still reach it.
   - Evidence: card 13 Intent, case 5, last sentence; card 5 Intent, the first `File` bullet.

3. **MEDIUM — D8 / Card 5 `Settle`: an action that keeps failing is only logged.**
   - A failed `RunAction`, such as a spawn error of `lym adopt check`, keeps the record, logs a Warn and retries every tick.
   - Nothing tells the orch or the operator. The held runs wait for a check that never starts, and the record names no hand step.
   - It needs a once-per-episode notice naming the hand step `lym adopt <subject>`. The dobby's other failures have one (dobby `doc.go`: "posts one notice … per episode").
   - The test row "a failed action runs again next call" asserts no notice.
   - Evidence: card 5 Intent, the Settle bullet "a failed action keeps the record for the next tick and logs a Warn"; card 6 says a `SettleFronts` error is "logged once per episode … and stops nothing".

4. **LOW — D2 / Card 11 `reconcileWarnings`: one fixed text for three different states.**
   - The card writes `run lym fabric pull in that worktree, then re-run lym adopt` for `rewritten`, `diverged` and `ahead` alike.
   - The reconcile already words a way forward per state. `diverged` on the code side gets `codeDivergedWayForward`, which is more than a pull. `ahead` gets none, and a pull cannot settle local commits that origin lacks, so the re-run warns again.
   - The warning should carry `branch.WayForward` when it is set, and fall back to `lym fabric pull` otherwise.
   - Evidence: `internal/fabricengine/originreconcile.go:318-334` (`stopWayForward`); card 11 Intent, the D2 paragraph.

5. **LOW — D5 / Card 10 `hubCheckDeps`: a subject worktree that does not resolve leaves no trace.**
   - When the pair does not resolve, the spawned `check <subject>` prints its error to `check.out` and writes no record.
   - Adopt's envelope still says the check started, and `lym beam status` shows no check for the subject. Nothing names a way forward, such as `lym beam start <subject>` or re-running `lym adopt <subject>`.
   - Evidence: card 10 Intent: "returns an error when it does not resolve".

Tool calls made:
- `sed internal/boardengine/ownerset.go:40-162`: to see whether a re-run's owner write lists already-owned slugs in `Recorded`, so that the ways forward naming a re-run of `lym adopt <subject>` start a new check. It does; no finding.
- `grep` for `BranchAhead`/`IsRunStatus`: the reconcile's per-state way forward and the board's run-status rule, which my context lacked.
- `grep` for `stopWayForward` callers: to locate the per-state way-forward function.
- `grep` for `stopWayForward` definition: same, narrower.
- `sed internal/fabricengine/originreconcile.go:318-365`: the per-state way-forward texts that finding 4 cites.
