```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked that the change set agrees with the sibling cards, criteria and cross-references it touches.

1. LOW. Card 13's Intent, the sentence "It is how the orch watcher (card 17) asks for the same switch; card 14's `Wait` asks its own engine directly."
   This is now stale: card 14's new load-hold case also asks the switch.
   Evidence: card 14 now says "In `readLoadTurn`, a load turn end … is asked of the switch with the run directory and the turn end before the load hold counts it". So `readLoadTurn` asks the switch as well as `Wait`, and card 13 still names `Wait` alone.

2. LOW. Card 5's ImpactSummary: "two wiring call sites behind told seams; Worktree-Create and adopt gain one logged, non-failing call after each pair is added".
   This no longer matches the F12 change: the `complete` branch also calls `pickAccount`, and that call does not come after an add.
   Evidence: card 5 now says "Its `complete` branch … calls it with the slug and `keepEntry` true before `reconcileTaskPair`". That is a third call path, and it runs when no add happened.

3. LOW. Card 12's ImpactSummary: "One tolerant field and one predicate reached by every running-record reader in the package; …".
   It does not mention the new edit outside the package: the `lym shuttle state` verb and its test, both added for F10.
   Evidence: card 12's Edit list now holds `internal/shuttlecli#shuttleCLI.stateCmd` and `internal/shuttlecli#TestStateCmd_ListsRunningRunsAndNarrowsByStrand`.

The other changed text agrees with its siblings:
- Card 15's settle at attach or at `Wait` past the bound matches card 11's new bound sentence.
- Card 17's `relaunchOrch(st, fresh)` is separate from `relaunchUnderAccount`.
- No other card references `PickAccount()` or `relaunchUnderAccount(true/false)`.
- The overview's PATTERN bullet still matches the decision record.

Tool calls:
- I read card 15's head and grepped cards 9, 13 and 17 for relaunch, load-turn and PickAccount references. My context held none of those cards except card 13, and I needed card 15's Wait settle to check card 11's new bound sentence.
