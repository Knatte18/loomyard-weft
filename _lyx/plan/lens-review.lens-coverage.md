```yaml
lens: coverage
outcome: reported
```

Coverage lens: I checked every decision, acceptance criterion and card-introduced branch for a carrying card and a test row that would fail if it broke.

1. **MEDIUM**, Decision 3 and Decision 6, criteria "runs the build step once, inside the first such tier's slots" and "under `dotnet` an empty diff … runs no build unless a listed `carry: false` tier opts in with `shared_build`". No planned row runs the build at run time on a `dotnet` empty scope. Card 11's only row for the build-only run is a `TestPlanTierRuns` plan row. Its `TestGateTest_Scenario` rows all run under the scenario hub's `go` scoper, where the "no test class" share never arises. Nothing reaches these paths: the build-only tier taking its slots, running the build and running no test command; an empty `dotnet` diff running no build; an empty `dotnet` diff with an opted-in `carry: false` tier running the build before that tier. Evidence: card 11 (`plan:internal/gatecli#execution.runBuild`), whose Tests name only `TestPlanTierRuns`, `TestGateTest_Scenario` and `TestGateReport`; card 8, whose `TestGateTest_Scopes` rows have no `build` key.

2. **MEDIUM**, Decision 8 against card 11. Card 11's build-only flag can wrongly cover a `carry: false` tier, so that tier runs no tests. Card 11 sets the flag on any `shared_build` tier whose share holds no test class. Card 9 says a `NoCarry` tier runs whole under that same share. Card 11 never excludes `NoCarry`, so an opted-in `carry: false` tier under a no-class seed would run only the build. That breaks Decision 8, "so neither … an inert-only or no-class diff can leave it unrun". No row combines `NoCarry` and `shared_build` under that share. Evidence: card 11's Intent on the build-only flag; card 9's `scopeInvocations` bullet.

3. **MEDIUM**, Decision 6, criterion "with the tier's timeout counted after it". No row checks that a tier's `timeout_min` starts after the build ends. The timeout row only checks a build held past its own bound. Evidence: card 11 Tests; `internal/gatecli#runTier`, which starts its clock when called.

4. **MEDIUM**, Decision 6, criterion "not at all when every opted-in tier is carried or has an inert-only or empty-diff empty scope". Only the "all carried" case has a row (the second `--all`). Neither the inert-only case nor the empty-diff case is checked at gate level with a `build` declared. Card 10's inert rows carry no `build` key. Evidence: card 10 and card 11 Tests.

5. **MEDIUM**, Decision 4, criterion "`{filter}` renders the tier filter alone for … a whole-running tier inside a narrowed `dotnet` call, a command lacking `{targets}`". Card 7 tests only the rendering given a list of class filter terms. Which invocations get terms is decided by `scopeInvocations` (card 8). No row checks that these invocations get no terms:
   - a command without `{targets}` under a narrowed `dotnet` share;
   - a base group that fell back to whole while another group narrowed;
   - a `carry: false` tier inside a narrowed `dotnet` call.

   Evidence: card 8's `TestPlanTierRuns` rows (narrowed, no test class, `--guards`); card 9's rows don't check terms.

6. **MEDIUM**, Decision 3, fallbacks "a git read fails or a source file under the `dir` cannot be read", and card 6's `scanTree` error branch. No row in `TestDeriveDiff` or `TestDeriveCards` reaches an unreadable `.cs` file, an unreadable directory, or a failed `git show` / `git diff`. Evidence: card 6 Tests.

7. **MEDIUM**, Decision 2, "skipping `bin/`, `obj/`, `.scratch/` and the lyx and VCS directories". No row puts a `.cs` file or `.csproj` under `obj/` or `bin/`. A scan that read them would add stray types or a second project to the test project's directory, and no row would notice. Evidence: card 6 `scanTree` and its Tests.

8. **MEDIUM**, Decision 3, `--card` fallback for a seed file "names a `.cs` file outside the `dir`". `TestDeriveCards` has no row for it. Its rows are the member glyph, the file seed, the inert drop, all-inert, a `.json` file, a deleted `.cs` file and an empty seed. Evidence: card 6 Tests.

9. **LOW**, Decision 1, "for the computed, `--base` and `--card` forms". The `dotnet` rows in `TestGateTest_Scopes` cover the computed and `--card` forms but not `--base`. Evidence: card 8 Tests.

10. **LOW**, Decision 3, "a changed path, by its new or its old name, is neither a `.cs` file nor inert". No row renames a non-`.cs` file to a `.cs` name or the reverse, so the old-name check is unreached. Evidence: card 6 Tests.

11. **LOW**, Decision 4, "A tier command without `{filter}` under the `dotnet` scoper runs each selected project with no class filter". No row covers it. Evidence: card 7 and card 8 Tests.

12. **LOW**, Constraints, "A whole pass of two `vstest` tiers … carries both". Card 8's carry row uses the stand-in command, whose tiers count with `go-json` in `scenarioConfig`. The criterion names `vstest` tiers. Evidence: card 8 Tests; `internal/gatecli#scenarioConfig`.

13. **LOW**, Decision 6, "Its command takes no placeholder, so the words after `--` never reach it", plus the build's `env` and `dir`. No row passes `-- args` and checks that the build invocation didn't receive them, or checks the build's working directory or `env`. Evidence: card 11 Tests.

14. **LOW**, Decision 6, the blocked tiers' "run record naming the build failure and its log". The scenario row checks that commands didn't run and that no `tier0` summary line was printed. It doesn't check the blocked tiers' run records or the stderr line naming the log. Evidence: card 11 Tests (`execution.reportBlocked`).

15. **LOW**, scope refusals "run first under either scoper". The refactor moves the plan read and its refusals into the new `readCards` function. No row checks the unknown-id refusal or the no-plan refusal under `scoper: dotnet`. The existing loop covers only `go` and `none` (`internal/gatecli/scopes_integration_test.go:289`). Evidence: card 8 Tests.

16. **LOW**, card 12. The skip "when the offline restore finds no test packages" names no check that detects that case, so the skip branch is unspecified and no row reaches it. Evidence: card 12 Intent.

Further tool calls: none.
