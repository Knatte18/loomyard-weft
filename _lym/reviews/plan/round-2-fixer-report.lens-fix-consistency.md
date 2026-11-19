```yaml
lens: fix-consistency
outcome: reported
```

fix-consistency lens: does the F16 change set agree with the sibling cards, criteria and cross-references it touches?

no findings

- Card 6 still agrees with card 14. Card 6's `Run func(...) (LiveRun, error)` keeps its error, and the revised sentence "A `Run` error, a spawn or wait failure" matches card 14's new line: "The `Run` seam returns an error for a spawn or wait failure alone".
- Card 6's `TestLiveGate` row "a `Run` error is returned with its way forward and no report is written" still holds.
- `LiveRun.Refused` keeps the meaning card 6 gives it, the refusal text set only on a refused call, and card 14 now fills it only from a refusal envelope.
- Card 6's step 3 reads started lines from the run's output, and step 4 reads failing lines and the clamp line the same way. Card 14 now defines that output as stderr followed by stdout, kept whole, so those reads still find their lines whichever stream prints them.
- No other card (`00-overview.md`, 01–20) names `hardenLiveRun` or `LiveRun`, and none claims the live call prints an envelope, so no twin was left stale. Card 1's "envelope error" refusals agree with the new refusal-only envelope reading.

Tool calls:
1. grep of `_lym/plan/` for `hardenLiveRun`, `envelope` and `LiveRun`: my context did not hold the other cards, so this was the only way to find sibling references.
2. Listed `_lym/plan/` and grepped `_lym` records for `envelope` outside the changed cards: to find other plan files and decision text that might repeat the old claim.
3. grep of `00-overview.md` and `01-gate-copies.md` for live, run, exit-code and stream claims: to check the Shared Decisions and the copies card's clamp and started-line output against the new stream reading.
