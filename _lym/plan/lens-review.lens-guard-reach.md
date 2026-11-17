```yaml
lens: guard-reach
outcome: reported
```

guard-reach lens: whether each guard, refusal, branch and test row in the plan's cards can be reached, and whether a guarded site recurs unguarded elsewhere.

**Finding 1: MEDIUM. Card 8 (`LookupSeat`).** The refusal of "a name whose builder failed" cannot be reached.
- Card 6's `Seats(env)` returns one `Seat` only for each table that built, plus one error per failed builder. That error names the builder, not any seat name.
- A seat's `Name` exists only on a table that built. So nothing maps a requested name to a failed builder, and such a name always falls into the unknown-name refusal.
- `TestReport` has no row for this branch.
- Evidence: card 6's Intent describes `plan:internal/shuttleparity#Seats` as returning seats "of each table that built" with "one error per builder that failed, naming the builder". Card 8 describes `plan:internal/shuttleparity#LookupSeat` with "A name whose builder failed is refused naming that builder's error".
- Fix: drop that branch. The unknown-name refusal should name the catalog's seat names and every builder error `Seats` returned, so a missing seat from a failed builder still points at that builder. Add a `TestReport` row with a failing fixture builder.

**Finding 2: LOW. Card 4 (`resolveUserSetup` skips the warning for `ErrHookUnrun`).** The check depends on exact text, and no card states that dependency.
- The skip works only because `pluginSessionStartContexts` stores the hook runner's error verbatim as `Reason: err.Error()` (`internal/shuttleengine/claudeengine/pluginhook.go:166`). The warning that prints it is at `internal/shuttleengine/codexengine/usersetup.go:39`.
- If `UnrunHooks` wraps `ErrHookUnrun`, the text no longer matches and the warning returns.
- Card 6 says `UnrunHooks` "returns `shuttleengine.ErrHookUnrun`" but does not say unwrapped. Card 4's row uses `fakeUserSetupSource`, which hand-sets the `Reason`, so no test runs the real `UnrunHooks` → `pluginSessionStartContexts` → `resolveUserSetup` path.
- Evidence: `internal/shuttleengine/claudeengine#pluginSessionStartContexts`, `internal/shuttleengine/codexengine#Codex.resolveUserSetup`.
- Fix: card 6 states that `UnrunHooks` returns `ErrHookUnrun` unwrapped. Card 5's `TestNewWithSeams_CodexTakesTheGivenSeams` (or card 6's parity test) asserts no omitted-hook warning when the runner is `UnrunHooks`.

**Outside this lens, for the coverage lens.** Codex implements no `RequirementSupporter`; only Claude asserts it (`internal/shuttleengine/claudeengine/capabilities.go:16`). Under `meetsAll` (`internal/shuttleengine/requires.go:94-101`), Codex therefore fails any spec with a `Requires` entry, `background-wake` included. Cards 6 and 7 skip every such seat and producer, not only fork seats: for example `internal/loomcli#driverSpec`, which requires background-wake. How many seats the parity test actually compares is worth checking against the acceptance criterion "for every seat".

Tool calls made:
1. grep for `ReadDir` over the run-directory root, the `OmittedHook.Reason` field and `RequirementForks` in `shuttleengine`: to check which root scans card 2's `manifests` directory passes through.
2. sed of every root-scan loop and of `pluginhook.go:120-175`, plus grep of Codex's omitted-hook warning: to check that the scans skip non-directories and that the hook runner's error text reaches `Reason` unwrapped.
3. sed of `enginesrun.go:64-90` and `requires.go`, plus grep for `RequirementSupporter` implementers: to check the engines-run scan and how `Unmet` mirrors `Start`'s requirement check.
4. sed of `rundir.go:250-380`: to confirm that card 2's `sweepOrphans` and `findRunByStrand` rows reach the branches they claim.
5. grep for run-directory-root readers outside `shuttleengine`: to check whether the `manifests` directory recurs unguarded at another scan site (none found).
