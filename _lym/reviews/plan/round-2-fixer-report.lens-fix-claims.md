```yaml
lens: fix-claims
outcome: reported
```

Task: check that the facts the fix's new sentences state about the code hold at HEAD.

no findings

The claims I checked:
- **Card 9, the writer rows.** A failed gate ends the step at `seatRun.watch` (`internal/seatengine/engine.go:130-135`), and a failed start ends it at `Engine.Run` (`engine.go:250-252`). The died, timeout, wait-error and Accept-error ends all go through the same `r.seat.Required` branch in `watch`. Neither writer site queues a line. The resume end is card 10's, which card 10's lines 36-37 confirm.
- **Card 18, the help text.** The quoted sentences "five modes", "Every mode but --require-approved also runs rename-tool-missing, pattern-changes-mismatch, …" and "The four flags are mutually exclusive" match `internal/loomcli/validate.go:164,176,191`. The five modes today are no flag, `--require-approved`, `--rework`, `--lens-review` and `--overlay`, run by four flags, so adding `--part` gives six modes and five flags.
- **Card 7, the cited decision.** `pattern-line-unprescribed` runs only when `pc.DecisionRecordPath` is set (`internal/loomshed/patternplan.go:38,97`). The cited "Added after Discussion" entry exists in the decision record.
- **Card 12, the blank lines.** The template has a blank line after the Step 3 heading and one after the three opening lines, before `### What a card is`, as the earlier `cat -A` output shows.

Further tool calls:
1. I read the row names of `TestEngine_Run_RequiredAdvisorEndsTheStep` and the `Required` sites in `engine.go`, to check the claim about which ends reach which site. My context did not hold the test rows.
2. I read `seatRun.watch` and the start-failure branch of `Engine.Run` in `engine.go`, to check the two-site claim and the nothing-queued claim. My context held only the `stepEnder` lines.
