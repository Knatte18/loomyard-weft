```yaml
lens: way-forward
outcome: reported
```

Task: way-forward lens, checking each refusal, stop or recovery step the cards add and whether the next step it names can succeed.

**Per refusal: next step, and whether it progresses**

- Card 1, `scoper` other than go/none/dotnet: `want go, none or dotnet` plus the `fixIt` clause (`lyx config gate` from the prime). Progresses.
- Card 1, `tiers.<tierN>.filter` on a command without `{filter}`: `fixIt`. The tier is rewritten whole, as today. Progresses.
- Card 1, `build.command` missing, `build.timeout_min` missing, `build.timeout_min` below 1: `fixIt`. `build` is an open map (card 1 `ConfigOpenMaps`), so a whole `--set 'build={...}'` can write it. Progresses.
- Card 1, `tiers.<tierN>.shared_build` with an empty `build`: `fixIt` (declare `build` or drop `shared_build`). Progresses.
- Card 1, an `inert_paths` pattern that `path.Match` rejects: no refusal, it silently matches nothing. No next step is named. Noted only; it is a silent guard, not a refusal.
- Card 2, `BuildTimeoutLine`: `lyx config gate --set 'build={...}'` from the prime, carrying every key and the raised `timeout_min`, with the role clause. Progresses.
- Card 2, `verifyGate.verdict` on a build timeout: the `build={...}` rewrite replaces the `tiers.tier<N>={...}` rewrite. Progresses, but the reason's first clause stays wrong (finding 3).
- Card 2, a landing build failure that is not a timeout: today's "fix forward on the task branch, then resume", plus the fixer stencil line on `build: true` (fix the compile error in either mode). Progresses.
- Card 6, every `Whole` fallback: no refusal. The call runs whole with the reason on stderr, so it progresses with no next step needed.
- Card 7, a quoting error in `filterArgument`: it returns the `quoteAll` refusal, "pass a value without a percent sign". That does not progress when the `%` comes from the tier's `filter` in `gate.yaml` (finding 4).
- Card 8, the `readCards` refusals (no plan, unknown id): texts unchanged. Progresses.
- Card 8, `--guards` under `scoper: dotnet`: the card says the message names only dropping `--guards`. The refusal is built through `configRefusal`, whose trailing clause still says "fix it with lyx config gate". That does not progress (finding 1).
- Card 8, the `selected no test class, so <tierN> runs whole` line: information only. Progresses.
- Card 10, an `allInertSince` read error: logged at Warn, and the scope stays as the scoper derives it. Progresses.
- Card 11, a failed build: the blocked tiers' stderr lines name the build log, the closing line names `build`, and the exit code is the build's. Next step: fix the compile error. Progresses.
- Card 11, a build timeout: `BuildTimeoutLine` (as card 2). Progresses.
- Card 11, a build whose shell cannot start or whose `build.dir` is missing: no refusal or way forward is planned (finding 2).
- Card 12, test skips: each names its reason. Not a refusal.

**Findings**

1. **MEDIUM** — consistency — card 8, `08-dotnet-scoper.md`, Intent bullet "That refusal's way forward is reworded per scoper".
   The `--guards` refusal under `dotnet` is promised to name only dropping `--guards`, because setting `scoper: go` makes a C# hub run whole.
   But today's refusal goes through `configRefusal`, which always appends `; way forward: fix it with "lyx config gate" from the prime` (`internal/gatecli/tiers.go:216`, called at `tiers.go:118`).
   Rewording only the format string still leaves a trailing `way forward:` clause sending the agent to `lyx config gate`, and no config change helps a C# hub.
   The card's exact-message test row would then pin a way forward that cannot succeed.
   The card should say the `dotnet` form is built without `configRefusal`'s config clause, with `way forward: drop --guards, then re-run the same command`.

2. **LOW** — design — card 11, `11-shared-build-step.md`, `execution.runBuild` bullet.
   `runBuild` runs `build.command` in `build.dir` through `runTiedChild`, but the card plans no refusal for a build that cannot start, such as a `dir` that does not exist or no platform shell.
   `exitCodeOf` would turn that into a plain failed build with exit 1 and no way forward naming the directory.
   For the tier run, the same case is a refusal naming "make the tier's dir an existing directory and put the platform shell on PATH" (`internal/gatecli/tierrun.go:166`).
   Card 1 validates no `build.dir`, so this state can occur.

3. **LOW** — consistency — card 2, `02-build-run-lines.md`, `verifyGate.verdict` bullet.
   For a build timeout, only the `tiers.tier<N>={...}` rewrite is replaced by `build={...}`.
   The reason's first step, "fix the hanging test on the task branch" (`internal/landingshed/verifygate.go:271`), is kept, but a build timeout has no hanging test.
   The fixer is pointed at tests for a stalled build.
   The build form should name the hanging build, such as a stalled restore or compile.

4. **LOW** — consistency — card 7, `07-filter-placeholder.md`, "A quoting error is returned as `renderTierCommand`'s."
   A tier `filter` from `gate.yaml` holding a `%`, which Windows `QuoteArg` refuses, gets the `quoteAll` refusal "pass a value without a percent sign" (`internal/gatecli/tiers.go:302`).
   That way forward addresses the caller's own words, but the value is in the hub's config, which the agent cannot pass differently.
   The step that progresses is fixing the tier's `filter` with `lyx config gate` from the prime.

Tool calls:
1. grep of `internal/gatecli/tiers.go`, `internal/gatecli/tierrun.go` and `internal/landingshed/verifygate.go` for the refusal texts — my context held these functions' text but not their line numbers, which the evidence needs.
