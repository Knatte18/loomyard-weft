```yaml
lens: leanness
outcome: reported
```

Task: review the decision record and support log through the leanness lens, looking for content to move or cut.

**Findings**

1. **MEDIUM — Acceptance criteria, the `Review:` bullet on D14.** It asks code review to check that "the probe run on OSL-1046 is recorded before stage 1's first task", but code review cannot check that from a diff. D14's steps are operator actions and note-splitting order, which no run's change carries out. Fix: cut the bullet, or reword it to the text a diff leaves, for example the split note or a doc carrying the order. The rest of D14 can stay because this run makes a Sketch, but its acceptance line does not belong.
2. **LOW — Constraints, the bullets for `PATTERN-config-layers`, `PATTERN-child-processes`, `PATTERN-engine-parity`, `PATTERN-cli-cobra` and `PATTERN-run-records`.** Each one restates a decision (D2, D3, D10, D7) or the `## PATTERN changes` section, for example "this record changes the entry (see PATTERN changes)" and "D10 gives every engine its own config directory". They add no constraint the decisions lack. Fix: cut them. Keep only the `PATTERN-shell-guard-table` line, which states a boundary no decision states.
3. **LOW — Constraints, the "creel grants `EXTERNAL` the right to inject…" bullet.** It duplicates D8's rationale word for word in substance. Fix: cut one of the two.
4. **LOW — Scope "Out" and D16.** The list of non-claims (kernel exploit, data through reachable hosts, landed lym on the host) appears in both places. Fix: keep it in D16 and leave only a pointer in Scope.
5. **LOW — Auto-mode assumptions.** Each bullet restates a decision and the support log's question ledger Q2–Q10, so the same pick appears three times. The section is required, but its bullets could name only the assumption ("the entry's config module became a file outside the hub") without restating the decision body.
6. **LOW — Open risks, the `config-keys-nested` line.** D9 already reads the cap "as today", so a key rename changes nothing in this design, and the line carries no risk. Fix: cut it.
7. **LOW — Notes for the plan writer, "it is about 200 lines".** This is a tally that will go stale. Fix: drop the count and keep "on the security boundary, tested on a real kernel".
8. **LOW — D4, the probe algorithm restated step by step.** This passes the completeness-before-leanness test, because it is the security requirement the plan writer must meet. Keep it, and treat the cited probe file as a reference rather than the requirement.
9. **LOW — Support log, Turn 3 and the question ledger.** They list the same picks twice. This is harmless because Plan-Write never reads the support log, but Turn 3 could carry only the trade-offs.

**Outside my lens:** the `PATTERN-child-processes` "changes" text was estimated at about 279 characters against the 280 cap and never measured, so it may trip `pattern-change-over-cap`.

**Further tool calls:** none.
