```yaml
lens: interactions
outcome: reported
```

Interactions lens: checking every pair of decisions that act on the same state, then each criterion and constraint against the decisions it touches.

**Pairs checked**
- D3 × D5 (pause step runs before the owner step, but pauses by "whose owner the write makes this machine's"): held. The pause set is computed from the write about to be made, as `holdTargets` does today with `writing`.
- D3 × D4 (a project held back by a local-commit stop ends the record `stopped`, so D4 keeps covering that subject): held.
- D4 × D2 `--discard` (a discarded record ends the cover): held. The discard bound is stated.
- D4 × D5 (a dobby under adopt is idle, and adopted runs are adopt-paused): held.
- D4 × D11 (D11 runs only when the subject is not under adopt; after the owner write stands, D11 applies): held.
- D5 × D8 (the green end no longer lifts the pause, so only a hand lift does; one legacy lift is bounded): held.
- D5 × the dobby owner lift (`recheckOwnerPause` is gated by `!run.Held`, and D5 maps the `adopt` entry onto that treatment): held.
- D5 × D7 (the beam step starts the dobby, and the adopted runs are paused): held.
- D6 × D5 × D4 ("no run goes on under the old lym"): **broken** for front tasks. See finding 3.
- D6 × D10 (a deploy that failed or was interrupted after the owner write, then re-entered through `lym s`): **broken**. See finding 1.
- D9 × D10 (D9's way forward for `interrupted` is "re-run the same `lym adopt`", but the `lym s` path re-runs `--here`): **broken**, same root as finding 1.
- D1 × D2 `--discard` (the worker takes `adopt.lock`; the discard is not said to take it): **broken**. See finding 2.
- D1 × D2 resume (the caller follows by pid, and the resume worker rewrites the pid): held.
- D2 restart at `reconcile` × D5 pause set × an operator `lym tender resume` made in between: **broken**. See finding 4.
- D2 flag merge × D7/D10 `--no-orch` (a resume can add `--no-orch` but never drop it): held. The criterion pins this, and `beam start` brings the orch up.
- D1 follow cap × D10 shortcut (`still running` exits non-zero, so the shortcut stops): held.
- D3 check step × D8 × Out list (the red fix task is filed after the pause step, and the dobby starts it): held. Out states this.
- D11 × D12 (archives appear after the owner write, and `--archives` is a read you can repeat): held.
- D7 × D10 (`--no-orch` in the shortcut; `beam start` starts the orch): held.
- Criteria D2, D3, D4, D5, D7 and D8 against their decisions: held.
- Constraint "pauses only unpaused slugs" × D2 restart: broken, finding 4.
- Constraint "starts no second live check" × D3: held.
- Constraint "writes the same owners" × D3: held.

**Findings**

1. MEDIUM · design · D10 × D6/D2/D9
   - Summary: `lym adopt --here` ends `owned_here` without reading the record. So `lym s`, run after an adopt that stopped or was interrupted once its owner write stood, skips the record's unfinished `deploy`, `beam` and `check` steps. It then goes on to `beam start` on the old lym, with no adopt check.
   - D10 says owned-here "spawn[s] no worker and read[s] or writ[es] no record". The record's D10 rationale covers only *another* subject's record, never the subject's own unfinished one.
   - D6 says a failed deploy "stops the record at `deploy`; the owner stands". D9's way forward is "the re-run of the same `lym adopt`", but on the documented entry point that re-run is `--here`, which now reads owned-here.
   - Purpose missed: "resumes after a break" (Goal).
   - Evidence: decision-record.md:135-138 (D6), :170-173 and :180 (D10), :165 (D9).
   - Today's chain stops at the first non-zero exit, and `--here` ends ok here: `internal/shortcutcli/shortcut.go:211-223`.

2. MEDIUM · design · D2 `--discard` × D1/D2 lock
   - Summary: `--discard` checks "whose worker is not alive" and removes the record, but takes no `adopt.lock`.
   - A discard racing a re-run can read the old dead record while a new worker takes the lock and writes `running`, then delete that fresh record.
   - The worker then runs with no record. Its caller loses the pid it follows, and D4's cover vanishes mid-adopt.
   - D2 states no bound for this race.
   - Evidence: decision-record.md:48-49 (worker takes `adopt.lock`, writes `running`) and :61 (`--discard` with no lock named).

3. LOW · consistency · D6 claim × D5 front-task exclusion × D4 end
   - Summary: D6 says "the runs stay paused, so no run goes on under the old lym". But D5 pauses only released tasks "that [are] not a front task".
   - D4's cover ends at the subject's owner write, so the subject's dobby, kept up by D4, may start an open front task under the subject after the owner write. That includes during a failed deploy, on the old lym.
   - The adopt fix task is a front task (`fronttask.ActionAdoptCheck`), but other front kinds are not excluded.
   - Evidence: decision-record.md:114 (D5 exclusion), :92 and :108 (D4 ends at owner write), :138 (D6 claim).
   - The fix task is filed through `fronttask.File` with `OnLand: fronttask.ActionAdoptCheck`: `internal/adoptcli/fixloop.go:269-279`.

4. LOW · design · D2 restart at `reconcile` × D5 × Constraints
   - Summary: a resume that adds `--keep-local` or `--drop-local` restarts at `reconcile`, so the `pause` step runs again.
   - The Constraint says a re-run "pauses only unpaused slugs". A run the operator already lifted with `lym tender resume` after the first `pause` step is unpaused, so the re-run adopt-pauses it again and stops work the operator started.
   - The record lists the slugs it paused (D5), but no decision says the re-run's `pause` step skips them.
   - Evidence: decision-record.md:57 (restart at `reconcile`), :118 (record lists paused slugs), :221 (Constraint).

Tool calls:
- `grep fronttask internal/adoptcli/fixloop.go`: to confirm whether the adopt fix task is a front task; the manifest held only `green.go`'s `FrontDeps`, not the filing.
