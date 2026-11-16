```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each review proposal the fixer carried out (fix-proposal lens).

Proposals checked: F1 option (a), F2, F3, F4, F5, F6, F7.

1. **MEDIUM.** F2, in the D2 reset probe and its acceptance criterion: "the probe finds a strand whose attempt has a recorded outcome but was not yet stopped … naming each way forward and stopping no strand".
   - The proposed probe does not find that strand. It removes it silently, so the criterion describes a test that cannot pass.
   - Evidence: an attempt gets its outcome recorded only after its Wait returns, and `finalize` (wait.go:1976-1982) writes a terminal outcome to run.json, such as done or timeout.
   - For a tracked, live strand with a terminal record, `dispositionCandidate` (attach.go:545-550) returns `verdictRespawnEligible`. `ProbeGated`'s not-found answer then removes it through `removeSupersededStrands` (attach.go:202-207).
   - So in this case the reset does not refuse and does not "stop no strand". The strand is stopped without `--stop-strand`, which contradicts the criterion and the "finds a run" refusal wording.
   - The strand still ends, so the hazard is closed. The refusal and criterion text for this case are wrong.

2. **MEDIUM.** F1 option (a), in D10: "The fixer carries no shuttle gates, so the row's `gates` entries ride the batch strands alone".
   - Concrete case: a row whose `gates` entry is a lint the verify gate does not run. The last batch passes it. The verify fails, the fixer commits `fix: …` that breaks the lint, and the verify re-run passes. The row returns Done with a row gate failing at HEAD.
   - This breaks the artifact's premise that the row's `gates` entries hold over the built work.
   - D7 already reasons that "a row `gates` entry needs a strand to run in" and refuses the no-spawn path for that reason. Under option (a), the fixer's commit is exactly such an unchecked change.
   - Option (b), or Go re-running the row gates after the fixer, would avoid this. The record states neither.

3. **MEDIUM.** F4, in D1: the restore is allowed "only when the task branch's HEAD is the end commit that version records for its last batch".
   - The bound misses the review's own second premise ("The restore also forgets an in-flight strand").
   - Concrete case: batch N+1 has begun and its attempt-1 strand is live but has not committed yet. HEAD equals batch N's end commit, so the restore is allowed.
   - The restored state has no record of the attempt. Re-entry has nothing in flight to attach to, and D12's attach covers only recorded in-flight attempts, so the row spawns a fresh attempt beside the live strand: two agents on one tree.
   - The bound also needs no live strand of an unrecorded attempt, for example the `ProbeGated` check by report path that the reset now uses.

4. **LOW.** F7, in D2: "every entry of `_lym/weaver/` moves into a UTC-stamped directory inside `_lym/weaver/`".
   - A second reset moves the first reset's archive directory into the new one, so archives nest without bound.
   - D9's PR rework archive of `_lym/weaver/` into `prior-generation/weaver` also carries every reset archive into the prior-generation slot that Describe's prior readers list. That breaks the stated property that Describe does not list them.

5. **LOW.** F3, in D6: "or `gatefixer.Await` warns that a fixer's stop failed … so D12's first step stops it again on re-entry".
   - D12's first step "stops the live strand of every attempt whose outcome state records". A fixer is recorded by fixer number and a `gatefixer` record, not as an attempt, so whether that step covers a fixer is unstated.
   - In that reading, a fixer whose stop failed is not stopped again on re-entry.

Tool calls:
- grep of `finalize` in shuttleengine: my context lacked how a waited run's record ends.
- Read of wait.go:1944-2013: to confirm `finalize` writes a terminal outcome, not running.
