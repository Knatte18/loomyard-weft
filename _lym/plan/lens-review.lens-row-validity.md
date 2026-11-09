```yaml
lens: row-validity
outcome: reported
```

Lens: row-validity. I checked whether each planned test row reaches its branch and fails when the code is wrong, and whether each new test duplicates an existing one.

1. **BLOCKING** — Card 7, `TestRunWorker` row "resumes at `deploy`", and the resume path from cards 6 and 7. The row cannot pass as written.
   - On a resume past `owner`, `stepRun`'s per-slug `outcomes` map starts empty, because no card says the resumed steps rebuild it from the record.
   - `runBeamSteps` does `out := outcomes[slug]` and then writes `out.BeamSteps` (internal/adoptcli/adopt.go:914, 927). For a beam in `Inputs.OwnerStood` that pointer is nil, so the worker panics before `done`.
   - The row's expectation "the record ends `done`" therefore fails. The criterion is real but neither card prescribes the code that makes it reachable.

2. **MEDIUM** — Card 6, `TestAdopt` row "a second run of the pause over a record listing a slug the operator resumed leaves it unpaused". The row cannot reach the skip.
   - In card 6 no record is persisted, and each `adopt()` call builds a fresh `stepRun` whose `Inputs.Paused` is empty.
   - A second `adopt` call pauses the resumed slug again, so the row's assertion fails. Otherwise it only passes by seeding `stepRun` directly, which `TestAdopt` does not do.
   - `TestPauseEntries`' "listed slug skipped" row already pins this. Drop the `TestAdopt` row, or move it to card 7's `TestRunWorker` over a seeded record.

3. **MEDIUM** — Cards 6 and 7, the fake world's `order`. The rows that assert which steps ran need fake changes no card states.
   - Card 6 says `order` shows reconcile, pause, owners, deploy, beamup and suite. Card 7's resume rows assert "no reconcile" and "runs the reconcile again before its owner write".
   - Today the `Reconcile` seam only sets `w.reconciled` and appends nothing to `order` (internal/adoptcli/adopt_test.go:135-141). `AddPair` appends nothing either (:144-150).
   - Card 7 also asserts no pair add on a resume (the D2 criterion names "pair add"), and no row checks it.
   - `world.deps` sits in card 6's Edit list, but the Intent never says that `Reconcile` and `AddPair` must append to `order`.

4. **MEDIUM** — Card 8, `followDeps` and `TestFollow`. Two rows need seams the card does not list.
   - The row "steps print as they end" needs an output writer, but `followDeps` lists only the record read, liveness, `exited`, the clock and the sleep.
   - The row "at the cap it returns `still running`, and with cap zero it waits on" needs the cap injected. `follow`'s signature `follow(deps, call, workerPID, since)` takes no cap and `followDeps` lists none.
   - As written, neither row can drive or observe its branch.

5. **MEDIUM** — Card 8, D1 and D7 at the verb level. No row reaches `Command`'s `RunE` mapping.
   - That mapping covers: exit 0 for `done`, non-zero for `stopped` or `interrupted`, `runsInOrch` setting `NoOrch` on the request, and `--discard` combined with another flag refused.
   - `spawnWorker` refuses under test, so `RunE` never gets past the spawn in any planned test.
   - `TestFollow` returns a record or an error and never an exit code. `TestRunsInOrch` tests the predicate alone, not that `RunE` sets `NoOrch` from it.
   - The `--discard`-with-other-flags refusal has no planned reaching test at all (rubric: "Reachability of a prescribed refusal").

6. **LOW** — Card 1, `TestPause` row "an `AdoptCaller` entry answers `AlreadyPaused`". It repeats an existing row.
   - The existing row "a request's Caller labels the added entry and adds none over a live entry of the scope" (internal/tenderengine/pause_test.go:777-793) already shows `AlreadyPaused` is caller-agnostic: a second `Pause` over an `OwnerCaller` entry answers already paused.
   - Card 1 changes nothing in `Pause`, so a new caller value adds no branch.

7. **LOW** — Card 1, `TestResume` row "Resume lifts its own `AdoptCaller` entry". It repeats an existing behaviour.
   - `Resume` lifts every live entry of the scope whatever its caller (tenderengine/resume.go:111-130), and `TestResume` already resumes slug entries of other callers (e.g. resume_test.go:344, 386).
   - Card 1 changes no `Resume` code, so the row cannot fail on any change in this plan.

8. **LOW** — Card 8, `TestWorkerCommand` in the new file `internal/adoptcli/worker_linux_test.go`. It largely repeats `TestDetachSetsSetsid`.
   - `TestDetachSetsSetsid` (internal/proc/proc_linux_test.go:302-313) already pins that `proc.Detach` sets `Setsid`.
   - The new test only re-asserts that for a command `workerCommand` built through `proc.Detach`. A whole new file for that is questionable; at most it belongs as a row beside `TestWorkerArgs`.

9. **LOW** — Card 8, `TestSpawnWorkerStartsNothingUnderTest`. It is a new top-level test for the test-binary guard, the same shape as `TestStartSuiteStartsNothingUnderTest` (internal/adoptcli/deps_test.go:69). One table over both spawners' under-test refusals would cover both.

10. **LOW** — Card 2, `TestStart_StartsTheDobbiesFirst`. The new `skipOrch` column needs assertions the table cannot make yet.
    - The table asserts only `wantFirst` and `wantWarning` (internal/orchcli/startdobbies_test.go:24-39).
    - Card 2's row asserts "the strand is neither launched nor its watcher spawned", which needs a launch or watcher observation the struct has no field for. The card does not list that new expectation field.

Tool calls beyond my context:
- Read `internal/orchcli/startdobbies_test.go`'s `TestStart_StartsTheDobbiesFirst` table: my context named the test but not its columns.
- Grepped `TestResume` and `TestCoverageOf` rows: I needed which callers the existing rows already lift and cover.
- Grepped `proc`'s `Setsid` test and `cmd/lym`'s `unboundedWaitFiles`: I needed whether `Detach`'s session is already pinned.
- Read `TestPause`'s caller row: I needed whether `AlreadyPaused` over a foreign caller is already pinned.
