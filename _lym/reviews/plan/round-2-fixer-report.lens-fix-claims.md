```yaml
lens: fix-claims
outcome: reported
```

Task: check that each claim the fix added or rewrote in cards 14 and 6 holds against the code at HEAD.

**MEDIUM: "which only a refusal prints" is false.** Changed text: card 14, the `Refused` sentence: "…parses as a JSON object with `ok` false and a non-empty `error`, which only a refusal prints, so build output on stdout never matches."
- Two failures that are not refusals also print an `output.Err` envelope (`ok` false plus `error`) on stdout and halt the call:
  - a guard step that fails to build (`internal/gatecli/run.go:590`, `runGuard`, `output.Err(e.out, err.Error())`);
  - a slot acquire that fails (`internal/gatecli/run.go:762`, "gate test: cannot acquire a gate slot: %v; way forward: re-run the same command").
- Under the card's rule both of these land in `Refused`. Card 6 then returns them as a hub-config refusal with no report, so the fixer is never re-prompted. That outcome is defensible, but the sentence claims a property the code does not have.
- The wording should say "which only a call that halts before a verdict prints (a refusal, a guard step or slot acquire failure)", or card 6's `Refused` wording should be widened to match.

The other added claims hold:
- A run that reaches a verdict prints no envelope. Tier outcomes go through `execution.report`, which prints the trimpath note and `gateslot.TierSummaryLine` to `e.request.stderr` (`run.go:680-683`). The only `output.Ok` on this path is in `detachfront.go:1009`, the slot-busy answer, which comes only without `--wait`.
- The started lines go to stderr. Card 1 (`01-gate-copies.md:47`) has `execution.report` print them, and that function writes to stderr.
- The refusal envelope is the last stdout line. `report` and `runGuard` halt the call right after `output.Err`, and `output.ErrFields` prints one line.
- A refusal exits 1 (`doc.go:1462`), so "exit 1 whose last stdout line is a refusal envelope" in `TestHardenLiveArgs` matches.
- Card 6 now says "a spawn or wait failure", which agrees with card 14's `Run` seam sentence.

Tool calls:
1. grep of `run.go`, `acquireSlot` and `detachfront.go`, to find which paths print `output.Err` and `output.Ok`; the context held only the `doc.go` lines and `run.go:580-592`.
2. grep for a started-line producer in `internal/`; the context lacked where started lines are printed.
3. grep of `_lym/plan` for the started-line definition, because `ParseStartedLines` is planned and not in the code.
4. Read of `01-gate-copies.md:38-51`, for card 1's started-line spec.
5. grep for the `execution.report` function, to find its stream.
6. Read of `run.go:666-695`, to confirm `report` writes the summary to stderr.
