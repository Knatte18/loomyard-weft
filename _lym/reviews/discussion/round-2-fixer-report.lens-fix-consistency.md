```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fix's changes agree with the sibling text they touch or depend on (fix-consistency lens).

- **LOW.** Changed text: D3, the new bullet "A reach turn's failing result goes to the writer alone…", in its last sentence, "the count of failing results since the last pass counts shuttle arrivals, not inner gate runs".
  Summary: the earlier D3 sentence that owns the attempt-budget rule was left as it was, so D3 now gives the counting rule in two places with two different units.
  Evidence: the reach-turn bullet still says "Each gate is told its entry's attempt budget and counts its own failing results since its last pass; when one more failure would reach the budget, it passes without the turn". The new bullet then says what that count is counted in. A reader of the first sentence alone counts inner results. The review's fix asked for the counting unit to be stated "in the same bullet" as the bypass. The twin edit is to say "failing arrivals" in the first sentence, or to move the arrivals clause there.

Everything else I checked agrees with the changes:
- **Goal and Scope:** the build order, and Scope In's split of the `mutation` step into its own line.
- **D10:** the new opening sentence, and its refusal now deferred.
- **D11:** the output now named as D9's findings; the template sentence is gone and the board note asks the D10 question.
- **D12:** the `mutation` acceptance moved to D10's later build.
- **Constraints:** the refusal list is narrowed to D6 and D8, and the tallies are dropped from tolerant-records and stencil-ownership.
- **Open risks:** the trial-order bullet is removed.
- **Criteria:** the D10 bullets are marked for the later build, D11 now prints D9's findings, and the Review and D12 criteria are reworded.
- **PATTERN-test-speed change and the plan-writer note:** both drop `mutation`.
- **Support log:** a Round 2 entry is added, and the rejected-alternatives and Q10 lines are updated.

No further tool calls were made.
