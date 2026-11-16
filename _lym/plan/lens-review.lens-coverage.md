```yaml
lens: coverage
outcome: reported
```

Coverage lens: I checked every decision, acceptance criterion and Intent branch of the plan for a planned test row that would fail if it broke. Defects only, below.

1. **BLOCKING** (D6, D8, D9; cards 12 and 10): a `FAILED` report's journal entry and the per-batch commit are lost.
   - Card 12 sends only a `recordable` verdict through `recordBatch`; a `failed` verdict climbs the ladder straight away.
   - The journal append (card 10) and the commit of `_lym/weaver/` with the plan both live in `recordBatch` alone.
   - So a `FAILED` report, a timeout, a lost strand and a last-rung stop append no journal entry and commit nothing.
   - D8 says failed attempts' entries are kept, and D9 says Go commits after each batch "done or failed".
   - Evidence: `_lym/plan/12-run-loop.md` (the `runBatch` bullets), `_lym/plan/10-batch-record.md` ("Done or failed, it records…").

2. **MEDIUM** (D4; card 9): the strand carries the `batch` gate, not the card gates, and no row pins its gate set.
   - D4 says the strand carries the batch's card gates and the row's `gates` entries.
   - `batchGateSpec` builds one `batch` entry plus the row's gates, so the operand is near the record's but not the same.
   - The criterion "the strand carries the card gates and the row's gates" has no row: `TestClassifyAttempt` is card 9's only test.
   - The `batch`-key-missing refusal is also unreached.
   - Evidence: `_lym/plan/09-batch-attempt-strand.md` (the `batchGateSpec` bullet and the test line).

3. **MEDIUM** (D1, acceptance "a test runs each named step and the row then proceeds"; card 17): no test runs the named steps.
   - `TestImplementerGuard` plants archived state in place of running `lym webster reset --to start` or `lym weaver reset`.
   - The third way forward, setting `tapestry.implementer` back, has no row at all.
   - Evidence: `_lym/plan/17-implementer-guard.md` (the test line).

4. **MEDIUM** (D3; cards 12 and 4): three D3 behaviours have no `TestRun` row.
   - The next batch partitioned from a rename-rewritten pending card.
   - A batch's card set and start staying fixed across its attempts.
   - The first begin recording `PlanWeftCommit`, which reset (card 14) depends on.
   - Evidence: `_lym/plan/12-run-loop.md` (`TestRun` rows).

5. **MEDIUM** (D3, D11, acceptance "only the run's last batch renders as the last batch"; card 8): `TestRenderBatchPrompt` has no last-batch versus non-last-batch row for the main-packages gate.
   - Evidence: `_lym/plan/08-weaver-stencils.md` (the test line).

6. **MEDIUM** (D6 acceptance): two loss and stop branches have no row.
   - A machinery loss of a re-entry attempt retrying once on the same rung (card 12 and 13 rows cover batch and fixer losses only).
   - A fixer whose `gatefixer.Await` warns its stop failed returning Stuck naming the guid and `lym reed remove`, with no next spawn (card 12 rows cover only the batch strand's failed stop).
   - Evidence: `_lym/plan/12-run-loop.md`, `_lym/plan/13-run-reentry-and-resume.md` (test lines).

7. **MEDIUM** (D12 first step; card 13): a failed stop of a recorded-outcome strand at entry has no row.
   - The Intent's step 1 returns Stuck naming the guid on that failure, but `TestRun_Resume` covers only the successful stop.
   - Evidence: `_lym/plan/13-run-reentry-and-resume.md`.

8. **MEDIUM** (D7; card 13): the verify re-entry row checks only that the verify reruns.
   - Nothing checks that a failure then gets one last-rung fixer and stops again.
   - The no-spawn path's clean-tree condition (a dirty tree must not record done) has no row either.
   - Evidence: `_lym/plan/13-run-reentry-and-resume.md` (`TestRun_Reentry`).

9. **MEDIUM** (D8 acceptance "failed attempts' included"; card 10): `TestRecordBatch` has no row showing a failed attempt's journal entry appended.
   - Evidence: `_lym/plan/10-batch-record.md`.

10. **MEDIUM** (D2 acceptance; card 14): `TestReset` misses four branches.
    - A probe error refusing.
    - A live `verify-fixer` strand refusing over a corrupt state (only `weaver` is rowed).
    - Webster's run directory left untouched.
    - `.lym/weaver/` cleared.
    - Evidence: `_lym/plan/14-weaver-archive-and-reset.md` (the test line).

11. **MEDIUM** (D9, D13 acceptance "a PR rework round archives `_lym/weaver/` into `prior-generation/weaver`"; "Describe reads Weaver's `summary.md`"; card 22): only the lister is tested.
    - The archive closure's Weaver slot, `DescribeTable`'s active `RunRecordPath` and its seam refusal have no row.
    - Evidence: `_lym/plan/22-describe-and-rework-archive.md`.

12. **MEDIUM** (D9; card 12): `TestRun`'s Done row does not assert `outcome.yaml` written done or the run-record commit, which merge-in's in-flight read depends on.
    - Evidence: `_lym/plan/12-run-loop.md`.

13. **MEDIUM** (D13 acceptance "a corrupt Webster state file under a Weaver run blocks neither a decision notice nor merge-in's warning"; card 23): merge-in has no corrupt-Webster row.
    - Card 21 covers the notice side only.
    - Evidence: `_lym/plan/23-merge-in-warning.md`.

14. **MEDIUM** (D13 acceptance "a renumber racing a Weaver batch begin is serialized by the state-mutation lease"; card 20): no row pins the lease directory.
    - The new rows use fake `acquireLease` and `loadState` seams.
    - No row shows that `newRenumberDeps` takes Weaver's scratch-dir lease when Weaver is active.
    - Evidence: `_lym/plan/20-renumber-reads-active.md`.

15. **LOW** (D11; card 12): the way-forward text `ResolveAtDispatch` gets under Weaver is unstated.
    - D11 bars Webster's rebaseline and restore-plan verbs, and no row checks the conflict text.
    - Evidence: `_lym/plan/12-run-loop.md` (the begin bullet).

16. **LOW** (D13; card 21): two notice branches have no row.
    - The `verify-fixer` strand getting the inline notice while Weaver is active.
    - `RecordDelivery` matching a fixer by its record's guid.
    - Evidence: `_lym/plan/21-decision-notice-weaver.md`.

17. **LOW** (card 6): two branches and one operand are loose.
    - No row for `Active`'s setting-error or unknown-value branch naming `lym config tapestry`.
    - "last recorded batch end commit" over a map keyed by batch number leaves "last" undefined; the record says the end commit of the version's last batch.
    - Evidence: `_lym/plan/06-active-implementer-seam.md`, `_lym/plan/05-weaver-state.md` (`BatchEnds`).

18. **LOW** (cards 15, 16, 17, 24, 25): stated branches with no row.
    - Card 15: the verb refusing outside a hub.
    - Card 16: a state or wiring fault mapped to Stuck.
    - Card 17: a zero `AnchorRoot` returned as a wiring error.
    - Card 24: the Webster note.
    - Card 25: the seam-refusal fallback.

19. **LOW** (D1/D2 attack surface; card 14): `Reset` skips refusals Webster's reset runs before moving the branch.
    - Webster's `PlanReset` refuses foreign dirty paths, a mid-merge tree and the wrong branch (`internal/websterengine/reset.go`, `PlanReset`).
    - Card 14 states no such bound for Weaver's reset.

I made no further tool calls.
