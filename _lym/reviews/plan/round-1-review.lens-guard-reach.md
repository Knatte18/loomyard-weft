```yaml
lens: guard-reach
outcome: reported
```

Task: guard-reach lens. For each guard, refusal, branch and test row the plan's cards add, I checked whether a real entry point can reach it.

**Per guard: reachable or not**

- **Card 2.**
  - Reachable: `Open`'s pass-over of a `BuildFailure`-only map, an out-of-range index and an unparsable map, each with a test row.
  - Reachable: an absent, unreadable or stale index builds the table in memory.
  - Not stated: what `Open` does when a map directory is absent. See F2.
- **Card 3.** Reachable: the index write's prime-only check, the moved-HEAD skip and the Warn on a failed write. The failed write is reached by the row whose index path is held by a directory.
- **Card 4.** Reachable: the guard's three failure branches, each reached by violating source. It is a scan test, so it needs no input of its own.
- **Card 5.** Reachable: each new key's refusal below 1 and the unknown-report refusal, each with a `TestLoadConfig` row.
- **Card 6.**
  - Reachable: the judged-nothing branches (red tier, outside a hub, a non-Go scoper, no parent branch), each with a row.
  - Reachable: the refresh skips for a fresh map, a dirty package, a dirty linked package, the budget and the prime, each with a row.
  - Reachable: `ReachReportMatches`' false branches for `Note`, `RedGate` and `Prime`, a test-file change, a function's text and an added function.
- **Card 7.** Reachable: none written, red tier, judged nothing, a match error and another tree. Two branches are wrong or untested; see F6:
  - A prime report: the reason it renders is wrong.
  - An unreadable report: no row.
- **Card 8.**
  - Reachable: the `WriterOnly` bypass on the entry's own gate; the re-run run through `RerunMark.Run` in `spawnAndJudgeAgain`; a nil `Rerun`.
  - Unmarked as the card intends: the live-record (restarted driver) path.
  - Not reachable in production: the bypass on the forked closure. See F5.
- **Card 9.**
  - Reachable: the turn, the listed-function skip, a changed hash, the budget pass and the re-run's no-count.
  - Also reachable: `shuttleengine/wait.go:1993` runs every entry's gate again at `OutcomeDone`. That evaluation repeats the last arrival's tree, so the listed-function check makes it pass. Benign.
  - Reachable: `ReachPendingOf` answering false for another key, a later commit, a judged-nothing report and an unparsable file.
  - Not reachable in production: the `Pending`-error branch. See F4.
  - Reachable: the record-write failure. It halts the gate, unlike the read and match failures. See F4.
- **Card 10.** Every refusal is reachable except the 200-commit cap: no named input reaches it. See F1.
- **Card 11.**
  - Reachable: no parent branch, a failed merge-base read, a subtest, an unmapped test, the cap of five and the budget drop.
  - Undefined: the "an `open` that fails" branch, and whether the `TestGateTest_Scenario` row reaches its "not in the coverage maps" line. Both depend on F2.
- **Card 12.** Reachable: the `Triage` section and its absence.
- **Card 13.** Reachable: the uncovered block, the logger, string-constant and embedded-file exclusions, and one mutant per line.
- **Card 14.**
  - Reachable: the three refusals, the non-Go note, an uncommitted file and the timeout and memory-cap unjudged rows.
  - No row: "not judged past `reach_budget_min`", and the `mutation_budget_min` bound, whose outcome is also unstated. See F3.

**Findings**

- **F1 — MEDIUM, consistency, `_lym/plan/10-reach-verb.md`.**
  - Summary: the range-over-cap refusal names no input that reaches it.
  - The card says each refusal is "reached by a `TestGateReach` row with the input named" (10:59). Yet the fixture has "a parent branch and three landings" (10:71), and the cap is a named constant of 200 (10:66).
  - No row can exceed the cap unless the test builds more than 200 commits, or the cap is injectable through `reachRequest` or `runReach`. The card states neither.
  - Fix: name the input, either a `limit` field on `reachRequest` that the row sets low, or a fixture with cap+1 landings.
- **F2 — MEDIUM, consistency, `_lym/plan/02-testreach-reader.md`.**
  - Summary: `Open` never says what an absent map directory does, yet card 11's branches depend on it.
  - Card 2 lists every pass-over case (02:38-46) but not a `sharedDir` or `worktreeDir` that does not exist. A task worktree with no anchor maps, or a hub whose `--map` never ran, is a common case.
  - Card 11 takes an error from `open` as "no triage line anywhere" (11:39). Its `TestGateTest_Scenario` row also expects the "not in the coverage maps" line in a hub where no map holds the test (11:72). That row passes only if `Open` treats an absent directory as empty.
  - Fix: card 2 states that an absent directory reads as empty and refuses nothing (`PATTERN-lym-files`: an absent cache refuses nothing), with a `TestReader` row for it.
- **F3 — MEDIUM, consistency, `_lym/plan/14-gate-mutate.md`.**
  - Summary: two budget branches of `lym gate mutate` have no test row, and one has no stated outcome.
  - Not judged "past `reach_budget_min`, run it again" (14:43) has no row in `TestGateMutate` (14:58-69).
  - The `mutation_budget_min` bound (14:44) says nothing of what happens to the mutants left past it (unjudged, or not run and counted), and has no row either.
  - Fix: state the outcome past `mutation_budget_min`, and add rows that drive both budgets through `mutateCmd`'s `minuteUnit`.
- **F4 — LOW, consistency, `_lym/plan/09-reach-turn.md`.**
  - Summary: the decorator's error handling departs from the advisory note's.
  - A failure to record the listed functions is returned as the gate's error (09:68), which stops a writer's verify gate over an advisory note. `ReachPendingOf` swallows its own read and match failures with a Warn, "so it never stops a gate (D3)" (09:74).
  - The "a `Pending` error is returned" branch, with its test row (09:93), cannot occur in production, because `ReachPendingOf` never returns an error.
  - Fix: Warn on the record failure and pass, or state why it halts.
- **F5 — LOW, consistency, `_lym/plan/08-rerun-mark-writer-only.md`.**
  - Summary: the forked-closure `WriterOnly` bypass and its test row (08:36, 08:46) guard a state no production path produces.
  - Card 9 decorates only the `verify` entry's `Gate` (09:79-81). `resolveGateEntry` builds that entry with no `Forked` closure (`internal/shedrecipe/entries_gate.go:148`).
  - D3 asks for the forked check, so this is no defect to fix. It is noted only as an unreachable guard.
- **F6 — LOW, consistency, `_lym/plan/07-reach-review-note.md`.**
  - Summary: the code review rubric's note misnames why a prime-written report does not count, and one reason has no test row.
  - Card 7 calls `matchesHead` "only for a report that judged" (07:54). A `Prime` report judges, but `ReachReportMatches` is always false for it (06:94), so the rubric line would say "written for another tree". D5 wants the line to say why.
  - The "unreadable" reason (07:53) has no row in `TestRenderReachReportNote` (07:67).
  - Fix: add a prime reason and rows for prime and unreadable.

**Tool calls**

- grep `internal/shuttleengine` for where an entry's `Final` closure is evaluated: I needed to know whether the reach turn also runs at shuttle's finalize, and my context lacked the run loop.
- `sed` `internal/shuttleengine/wait.go:1990-2025` and grep for `evaluateGate(true)`: I needed to confirm that every entry's gate runs again at `OutcomeDone`, which my context did not hold.
