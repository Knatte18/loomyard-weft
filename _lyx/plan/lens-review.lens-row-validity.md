```yaml
lens: row-validity
outcome: reported
```

Lens: row-validity. I checked whether each planned test row reaches the branch it claims, would fail if the code were wrong, and is needed. I found 8 defects.

1. **BLOCKING** (Card 12, `TestGateTest_DotnetFixture`): the fixture repo has no ignore rule for `bin/` and `obj/`, so the computed scope falls back to whole and the `--filter` assertion fails.
   - Evidence: `internal/testkit/plankit/plankit.go:102`. `Repo` only writes files: no `git init`, no `.gitignore`, no exclude.
   - The card's offline restore and any build write `obj/` (and `bin/`) into the tree before `--base` derives.
   - Card 6 says `impactset#ChangedPaths` counts untracked files that are not ignored, and Decision 3 says a changed non-`.cs` path that is not inert runs whole.
   - So `obj/project.assets.json` turns the run into a whole run. The assertions "`--filter` holding `CircleTests`" and "counts are `CircleTests`' alone" then fail.
   - Fix: the fixture writes a `.gitignore` for `bin/` and `obj/` (or an `inert_paths` entry for them), and the card says the repo is `git init`'d through `gitkit`.

2. **MEDIUM** (Card 8, `TestGateTest_Scopes`, the narrowed-pass green row): "a narrowed pass … writes no green entry" cannot fail if the code is wrong.
   - Evidence: `internal/gatecli/green.go`, `openGreen`. A dirty worktree returns nil, so nothing is recorded.
   - The `dotnet` rows change `Circle.cs` through `edit` (`internal/gatecli/scopes_integration_test.go:85`), which leaves the tree dirty.
   - A pass on a dirty tree writes nothing whatever `recordPass` does, so the row proves nothing about the `partial` rule.
   - Fix: commit the `Circle.cs` change, run `--base <prior commit>` on the clean tree, then assert that `green.json` holds no entry and that a later `--all` runs the tier.

3. **MEDIUM** (Cards 8 and 10, the computed `dotnet`, `go` and `none` rows in `TestGateTest_Scopes`): if these rows sit after the committing rows, they fall back to whole instead of narrowing.
   - Evidence: `internal/gatecli/scopes_integration_test.go:368` and later. Those rows commit `lib/lib.go`, `other/other.go` and `docs/note.md` to the pair, and the reset at `:428` covers only one row.
   - A computed scope diffs from the parent merge base (`:466`), which then holds the committed `.go` changes.
   - The diff then contains `.go` paths, which are non-`.cs` (so `dotnet` falls back to whole) and not inert (so the "runs no listed tier's command" rows run tiers).
   - Neither card says where its rows go.
   - Fix: put the computed and inert-only rows before the `:368` block, as the Markdown-only row at `:174` is, or have them use `--base HEAD`. Only card 10's carry rows belong after the committing rows.

4. **MEDIUM** (Card 2, `TestMergeIn_PlanVerifyFailureRecord`, the build row): the "no `Tests`" assertion cannot fail.
   - Evidence: no test file in `internal/landingshed` sets `failingTests`.
   - The fixture wires no `failingTests` seam (`newMergeInFixture`; `verifyGate.recordFailure`, `internal/landingshed/verifygate.go:82`, calls it only when it is non-nil), so `Tests` is empty for every log.
   - Fix: the row sets `fx.m.gate.recorder.failingTests` to return a test, then asserts that `Tests` is empty when the log's first failure is the build line.

5. **MEDIUM** (Card 11, acceptance criterion "a closure that selects no class … runs the build step once, inside the first such tier's slots, failing when the build fails"): only the planner is tested.
   - Evidence: card 11's tests cover `buildOnly` in `TestPlanTierRuns` only. The `TestGateTest_Scenario` build rows use `--all` and hand-built shares, never a `noTests` share.
   - No row drives a `buildOnly` tier through `execution.runListed` to show it acquires slots, runs the build, runs no test command, and fails when the build fails.
   - Card 8's `dotnet` rows cover no no-class case either.
   - Fix: add a `TestGateTest_Scopes` row with `dotnet`, a build and a `shared_build` tier, and a change to a type no test class names. Assert one build record, no tier invocation, a non-zero exit when the build exits non-zero, and that a tier not opted in runs whole with the "selected no test class" line.

6. **LOW** (Card 11, the `TestGateTest_Scenario` build rows): the helpers these rows need are not listed on the card.
   - "the build's run record and log are named with `-build-`" needs the build summary line's log path, but `summaryLogPathOf` (`internal/gatecli/test_integration_test.go:337`) matches only `gate test: tier<N> ` lines.
   - The held-build timeout row leaves a sleeper grandchild. `tierRecords` drops `Build` records (`:267`), so the existing cleanup pattern (`scopes_integration_test.go:274`) never kills it.
   - Fix: list a build-line log-path helper on the card, and have the row clean up from `records`.

7. **LOW** (Card 6, Decision 3's fallbacks): no `TestDeriveDiff` or `TestDeriveCards` row reaches "a source file under the `dir` cannot be read" or "a git read fails", so the error path of `scanTree` goes untested.
   - `internal/impactset/guardscan_integration_test.go:73-81` already has `makeUnreadable` and `skipAsRoot` helpers for exactly this.

8. **LOW** (Card 12, `TestGateTest_DotnetFixture`): the card says the test "commits the fixture" but puts it outside every hub on `plankit.Repo`, which does not init git.
   - Evidence: `internal/testkit/plankit/plankit.go:102`.
   - The card should name `gitkit.Git(…"init"…)` and list `gitkit` under Uses, as card 6 does for `TestDeriveDiff`.

Tool calls made:
1. `grep` of `watchdogline_test.go`, `runlog_test.go` and `plankit`: to confirm `TestWatchdogLine` has a `FindTierTimeout` table and `plankit` has `Repo` and `Language`, neither of which my context held.
2. `sed` of `green_integration_test.go` plus a `grep` of `hubforge` exclude: to see `TestGreenRunRecordPass`'s structure and how hub pairs hide untracked state, which my context lacked.
3. `sed` of `plankit.Repo`: to check whether it inits git or writes ignores, needed for finding 1.
4. `grep` of the `landingshed` failing-tests seam and `writeVerifyLog`: to check whether the Merge-In fixture wires `failingTests`, needed for finding 4.
