```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's code correctness (fix-code-correctness lens).

I ran each changed branch with one concrete input and found no BLOCKING or MEDIUM issues. The new tests I traced fail with their defect in place. The two findings below are both LOW.

- **LOW, `internal/shedrecipe/entries_harden_test.go`, `BouncerRowAutoAcceptStopsAtTheCampaignsRedLiveReport`:** The tests resolve only `auto_accept_setting`, through a fake `env.Setting`.
  - Nothing pins `reviewer_fan_setting` or `fixer_fan_setting`, which F2 also named. Nothing ties the recipe rows' `hardener.*` references to the settings the `configreg` row reads.
  - Evidence: `contracts/recipes/hardener-recipe.yaml` lines 58, 59, 84, 85 and 120 hold those references. The new `configreg` row reads `hardener.auto_accept` and `hardener.reviewer_fan` from `hardener.yaml` only.
  - Partial coverage: dropping the `fixer_fan_setting` wiring or changing a recipe reference to `loom.*` still passes every test.
- **LOW, `internal/gatecli/run.go`, `exe.green = greenForCall(green, request)`:** `TestGreenForCall` pins the helper, not its call site.
  - Replacing that line with `exe.green = green` leaves every test green.
  - Nothing at the call level checks that a copies call leaves `exe.green` nil.

**Traces that held:**
- **`startHead`:** the first call reads `Head` and writes `harden-head-N.json`, and a resumed call reuses it. A `Head` failure returns `read the head: …` wrapped once by `fileError`. `ResumedFixRowKeepsTheHeadItFirstRecorded` would see `head0` instead of `head-first` without the fix.
- **`NewLiveGate` deferred line:** a red mark with no failing line yields the single report line, and the new rows would see `nil` without it.
- **`CheckCampaign` empty lists:** a nil or empty `Reviewer` or `Fixer` is refused before `reviewerRepeat`, and `validCampaign` now carries a `Fixer`. `Campaign.models` already handles empty lists.
- **`runWithSlots` and `copying`:**
  - A guard-step run in a copies call has `copiesRun` 0 and takes its tier's own slots.
  - With the defect it would run twice, so the `GuardStepRunsOnceWhateverTheCopies` row fails.
- **`ArchiveDeps.AnchorRoot` and `Campaign.AnchorPath`:** `archiveDeps` passes `AnchorPath` and `LiveGate` still uses `WorktreeRoot`. `TestCampaignArchivesUnderTheAnchorPath` fails if `archiveDeps` regresses.
- **`Teardown.lymBinary`:** `New` sets it. The unit tests that build a `Teardown` literal replace `clearSandbox`, so none reaches a nil `lymBinary`.
- **`TestRunTierCopies_BuildsThePrebuiltLymOnce`:** a build per copy would count 2 and fail.

No further tool calls were made, because my context already held every changed file.
