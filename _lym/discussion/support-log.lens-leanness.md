```yaml
lens: leanness
outcome: reported
```

Lens: leanness. I looked for content in the decision record that belongs in the support log or can be cut.

**Findings**

1. MEDIUM — `## Auto-mode assumptions` repeats the support log's question ledger.
   - Each bullet restates a pick that a Decision already records: location (D1), sweep (D6), JSON (D1), error policy (D3), race bound (D6).
   - The "not X" halves are rejected alternatives, which the compaction rules keep out of the decision record.
   - Fix: cut each bullet down to a pointer, such as "D1, D3, D4, D6: picked without an operator; see the ledger". The section is required, so it cannot go entirely.

2. MEDIUM — `## Open risks` repeats D6's two bounds.
   - The owner-pause race and the unregistered hub are both stated, with their ways forward, under D6.
   - Fix: drop both lines, or keep only a risk D6 does not state.

3. MEDIUM — the closing D6 rationale holds a rejected alternative: "`lym adopt` is not used because it reconciles, holds runs and starts an adopt check".
   - The compaction rules say `## Decisions` carries Decision + Rationale only, with rejected alternatives in the support log, where this already appears.
   - Fix: keep the first clause (one name on the board for every machine and for the Slack and cross-machine work that follows) and cut the `lym adopt` clause.

4. LOW — the D1 rationale argues against the `configengine` machine layer.
   - That is a rejected alternative, and the support log already holds it.
   - Completeness-before-leanness: the plan writer needs the constraint that the file is no `configengine` module, and `## Constraints` already carries it under PATTERN-config-layers.
   - Fix: trim the rationale to "one value for every hub, readable outside a hub; JSON keeps the leaf on the standard library".

5. LOW — the D4 example list (`OSL46`, `osl.46`, `osl:46`, `osl_46`) is repeated word for word in an acceptance criterion.
   - Fix: keep the examples in the criterion only. The rule sentence in D4 is enough.

6. LOW — the D3 bullet "It also exposes the former names … for D5 and D6" is plan-level API detail.
   - D5, D6 and D7 already need the former names, so the plan writer will infer it.
   - The same goes for "The file name is one named constant" in D1: PATTERN-code-conventions already requires it.
   - Fix: cut both.

7. LOW — `## PATTERN changes` lists two "changes" bullets whose new text equals the current overview line.
   - The real changes are background-only, and they sit in `## Notes for the plan writer`.
   - A plan writer reading only the bullets cannot tell what changes.
   - Fix (emphasis, not exclusion): state the background change in each bullet's context, or keep the bullets and cross-reference the notes. If the change is background-only, check whether the bullets are needed at all.

8. LOW — the Slack bullet in `## Constraints` is long: three sentences of research on keeper internals.
   - It is the evidence for a fact the design rests on, so it belongs in the record.
   - It is wordy, though: "existing channels stay by id and are archived under their old name; new ones take the new name (`internal/slackengine/keeper.go`, `retireEntry`)" carries the same content.

9. LOW — `## Scope` In repeats the Decisions one-to-one: each bullet maps to one of D1 to D8.
   - This is acceptable as a summary.
   - The Out bullet "Slack channel names … see Constraints" is useful and stays.

Further tool calls: none.
