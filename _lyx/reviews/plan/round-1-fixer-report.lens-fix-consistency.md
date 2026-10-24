```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked whether every changed sentence still agrees with the sibling cards, rows and cross-references it touches, and whether any twin was left unchanged.

1. LOW. Card 7's ImpactSummary (`_lyx/plan/07-filter-placeholder.md:31`) and Tests paragraph were not updated for the new tier-number parameter.
   - The fix re-signed `renderTierCommand(number int, tier gateslot.Tier, targets, terms, args []string)` and `filterArgument(number int, ...)` (lines 4, 8 and 17).
   - The ImpactSummary still says the function is "re-signed here with nil terms for every invocation today" and never mentions the number.
   - `TestRenderTierCommand` is said to gain only "a `terms` column and the tier's filter". It needs a tier number as well, because the new budget Info log and the new quoting refusal both name the tier.
   - Evidence: the card 7 lines 4, 8, 17 and 31 quoted above.

Every other change agrees with its siblings:
- **Card 2 `BlockedBy`** matches card 11's `reportBlocked` bullet and its Scenario row.
- **Card 2 build-timeout reason** matches its `TestVerifyGate_CheckMapsVerifyResult` row.
- **Card 6 git-read `Whole`** matches line 36 ("a git read ... fails") and line 39 (the cancelled `ctx` is the only error).
- **Card 8 C# tree** satisfies card 5's nearest-`.csproj` rule, and card 11's `Unused.cs` row reaches `NoTestClass`.
- **Card 8 `partial` change** matches card 9 line 38 ("a narrowed `dotnet` share, in the last with no terms and `partial` unset").
- **Card 8 `dotnet` `--guards` text** matches card 11's new Long-help bullet.
- **Card 10's `go`/`none` inert rules** match its own test rows, line 39.
- **Card 11's start refusal** fits `tierResult.refusal`, which `runBuild`'s `tierResult` return can carry (`internal/gatecli/tierrun.go:38`).

Tool calls:
- grep of the plan cards for twins of the changed identifiers: my context held the cards only as the fixer edited them, not a cross-card index.
- Read of `tierResult` and the head of `TestRenderTierCommand`: my context lacked the `tierResult` fields, which decide whether `runBuild` can carry a refusal.
