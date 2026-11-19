```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the F16 fix covers every place in the plan that assumes a gate envelope from a passing or failing run.

No findings.

The F16 premise covers every plan site that reads the `lym gate test` result through a JSON envelope. Grepping `_lym/plan` for `envelope`, `hardenLiveRun` and `LiveRun` turns up these sites:
- Card 14's glyph (line 10), the Intent paragraph and the `TestHardenLiveArgs` rows are rewritten.
- Card 6 line 36 is rewritten. Its line 20 `Run` signature and line 22 `LiveRun` fields agree with the new text, so they need no change.
- Card 1 line 33 ("Refusals, each an envelope error") describes refusals only, which still print an envelope, so it stays correct.

No other plan card mentions a JSON or gate envelope. Card 1 never says which stream its clamp line and started lines go to. Card 6 reads both from the output, which is now stderr followed by stdout, so either stream works.

Unchecked: none.

Tool calls:
1. Grepped `_lym/plan` for envelope, `hardenLiveRun` and `LiveRun` mentions outside the edited lines, to find the full set of affected sites, which my context lacked.
2. Grepped card 1 for the clamp and started-line streams, and `_lym` for other JSON-envelope mentions, to confirm card 6 can still read them from the new combined output; my context did not hold card 1.
