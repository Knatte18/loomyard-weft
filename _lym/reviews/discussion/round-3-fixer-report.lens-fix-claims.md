```yaml
lens: fix-claims
outcome: reported
```

Lens: fix-claims. I checked each bound, behaviour or code fact that the fix added or rewrote against HEAD.

**[MEDIUM] "so a budget-halted perch halts on its next Stuck" (D2, new bullet on a goto from a perch that has not settled)**
Summary: after the goto, settling the recorded round again never returns a Stuck that Shed counts, so the claimed halt does not follow.
Evidence:
- The round that hit the budget is settled again with the soft-cap window open (`budgetReached`, `internal/shedadapters/bouncer.go:1348-1349`).
- That settle ends in one of three ways:
  - An escalation, when an escalation record exists (`bouncer.go:1358-1359`). That is Awaiting, not Stuck.
  - Through `softCapRule`, an automatic accept (`bouncer.go:1404-1408`).
  - Through `softCapRule`, an escalation or an automatic continue whose Stuck is marked `BudgetExempt = true` (`bouncer.go:1440`).
- `episodeStuckCount` skips a `BudgetExempt` Stuck and never counts it (`internal/shedengine/run.go:672`).
- So the perch continues past its budget or escalates. It does not halt on Shed's budget at its next Stuck.
- The claim came over from the reviewer's proposed fix.

**[LOW] "since they carry producer `Raddle-File` and `RoutedTo` `Raddle-Write`, an entry no verb writes" (D4, `freshRaddleEntry` correction)**
Summary: only one of the two goto rows carries `RoutedTo` `Raddle-Write`.
Evidence: `internal/loomcli/raddlewiring_test.go:41` (`GotoIntoTheRow`) carries `RoutedTo: loomshed.NameRaddleWrite`. Line 42 (`GotoElsewhere`) carries `RoutedTo: loomshed.NameRecordWrite`. Both carry producer `Raddle-File`, an entry no verb writes.

These claims hold against the code:
- the `designWrongKeys` and soft-cap wording, including that a missing tag counts and that LOW and NIT entries never count (`bouncerfiles.go:282-293`, `bouncer.go:1403-1409`);
- `dependencyOrder` keeps proposal order for ties;
- `Call`'s two filed-as-own tests and that the store-done path skips straight to `finish` (`raddlefile.go:137-183`);
- `EndRaddleEntry` would mark the project done;
- `Goto` and `Relaunch` write a goto entry whose producer is the target and which has no `RoutedTo` (`goto.go:140`, `:263`);
- a seed starts no generation without a Subject seam;
- `lym loom resume` grants one more budget (`run.go:648`);
- `lym loom circling accept` on an escalation raised again accepts through `acceptRound` (`bouncer.go:1270`), so the new proposal is filed with no round having reviewed it;
- `lym batten cancel` ends the run and clears the claim without removing the entry.

Tool calls:
- grep for `softCapRule` and `settleUndecided` in `internal/shedadapters`: my context did not show where they are.
- Read `bouncer.go:1330-1449`: my context lacked the soft-cap and undecided settle code that the D2 and D4 claims rest on.
- grep for the cancel functions in `internal/battencli`: my context did not show where `lym batten cancel` is implemented.
- Read `cancelcmd.go:48-147`: my context lacked what cancel does to the entry, which the new D6 way forward claims.
