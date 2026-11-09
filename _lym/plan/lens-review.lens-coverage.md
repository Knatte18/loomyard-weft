```yaml
lens: coverage
outcome: reported
```

Coverage lens review of the adopt-worker plan (cards 1–10 against the decision record).

**MEDIUM**: D1 and D2, card 8 (`Command` RunE). None of the `RunE` paths card 8 adds has a test that reaches them:
- follow-without-spawn on a live same-subject worker (D2: "a re-run while a live worker runs follows it and spawns nothing");
- exit 0 for `done` and non-zero for `stopped` (D1);
- `runsInOrch` setting `NoOrch` on the request (D7: "from an orch strand… starts the dobby and no orch");
- the refusal of `--discard` combined with a subject or flag.

`TestJudgeStanding`, `TestFollow` and `TestRunsInOrch` pin the pieces but not their wiring in `RunE`. A RunE that spawned on `follow`, ignored `runsInOrch` or exited 0 on `stopped` would pass every planned row.
Evidence: card 8's test list (TestFollow, TestDiscard, TestRunsInOrch, TestWorkerArgs, TestSpawnWorkerStartsNothingUnderTest, TestWorkerCommand) has no row driving `internal/adoptcli#Command`. `spawnWorker` refuses under a test binary, so the spawn path needs a seam to be reachable.

**MEDIUM**: D1, card 8 `follow`. The branch "a record not started later than the caller's own start is not followed" is in the Intent but no `TestFollow` row reaches it. The planned rows cover a later-started record with the same subjects and one with other subjects, never an earlier-started one.
Evidence: card 8, `follow` bullet 3 and the `TestFollow` row list.

**MEDIUM**: D3 acceptance "a project held back by a local-commit stop ends the record `stopped` while the other subjects reach `check`". Card 6's row asserts only `adopt`'s error and the `suite:` order; no record exists until card 7. Card 7's `TestRunWorker` has no held-back row asserting `State == stopped` at `done` with the way forward naming `--keep-local` or `--drop-local`.
Evidence: card 6, Tests bullet "A held-back project row…"; card 7, the `TestRunWorker` row list.

**MEDIUM**: D3 and card 6 `stepPause`. "A failed pause update stops the record at `pause`, naming the re-run" has no row. A `world` `UpdatePause` error seam is not listed either.
Evidence: card 6, `stepPause` last sub-bullet; card 6 and card 7 test lists; `internal/adoptcli#world.deps` UpdatePause has no error field.

**MEDIUM**: D5 and D8 acceptance "a green adopt check… leave every adopt pause and legacy `adopt-hold` entry standing" and "the green end starts no pass, brings up no dobby or orch and runs no `lym loom resume`". Card 4 argues this holds by construction ("greenDeps keeps no seam that lifts, starts or resumes") and plans no row. The rubric counts a criterion argued without a test as a finding.
Evidence: card 4 Intent, last test paragraph.

**MEDIUM**: D5 acceptance "a hibernate wake leave[s] every adopt pause… standing". Card 1 argues it holds without a test. The claim is true: `hibernateengine/wake.go:104-125` (`holdRuns`) only adds entries, and `reconcileAtWake` touches only the request's own hub entry. Still, no row pins it.
Evidence: card 1 Intent ("The hibernate wake only adds `OwnerCaller` entries and lifts none"); `internal/hibernateengine/wake.go:50-125`.

**MEDIUM**: D5 acceptance "an adopted beam's own-scope `owner` entry… which… the run report names". `TestPauseEntries` covers the conversion and `TestResume` the lift. No `TestRunReports` or `TestAdopt` row shows a converted beam slug in `runs` as `paused_by_adopt`, which depends on `Paused` joining the candidates.
Evidence: card 5's TestRunReports rows all use `t1`; card 6's TestAdopt pause rows don't name a beam slug in `runs`.

**LOW**: D2, card 7. "A fresh worker writes the record `running` at `reconcile`, with the resolved subject set, before any step runs" has no row. An assertion through the `Save` fake at the first step's start would pin it.
Evidence: card 7, runWorker step 4 and the TestRunWorker list.

**LOW**: Card 7, restart at `reconcile` over a record that already lists `reconcile` and `pairs` as done. The Intent keeps the done steps, so re-running them appends a second `StepRecord` for each. It doesn't say whether `envelopeOf` merges or replaces duplicates, and no row covers a merge with duplicates.
Evidence: card 7, steps 4 and 5; card 6, `runSteps` "appends a `StepRecord`".

**LOW**: D9, card 9. No row covers a read error of `adopt.json` joining the verb's failures.
Evidence: card 9 Intent ("A read error joins the verb's failures") and its test list.

**LOW**: Card 8 `discard`. "No record" reported as nothing removed has no row; `TestDiscard` lists only interrupted, lock held and done.
Evidence: card 8, TestDiscard row.

**LOW**: Card 8 `followCap`. Neither the keep-warm wiring nor its zero for an empty strand has a row; `TestFollow` takes the cap as a value.
Evidence: card 8, followCap paragraph and the test list.

**LOW**: Card 5 `ownerHaltResume`. No row shows it returning "" for a run blocked for another reason; the existing halted rows check by substring, so a stray clause would pass.
Evidence: card 5 Intent; `internal/adoptcli#TestRunReports` row `HaltedForBlockedWithItsReason` (`wantInLine`).

**LOW**: D1 "registered under `runproc.HubOwner` with an adopt role". Card 8 plans no row for the registration; `spawnWorker` refuses under test, so it is unreachable. Review is the only check, as `PATTERN-child-processes` allows, but the plan doesn't say so.
Evidence: card 8, `spawnWorker` paragraph and `TestSpawnWorkerStartsNothingUnderTest`.

Calls made:
- I read `internal/hibernateengine/wake.go:45-125` to check card 1's claim that a hibernate wake lifts no adopt pause; my context held only a summary of that code.
