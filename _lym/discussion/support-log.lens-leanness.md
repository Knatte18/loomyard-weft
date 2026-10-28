```yaml
lens: leanness
outcome: reported
```

Lens: leanness. I looked for content in the decision record that should move or go, and applied the completeness-before-leanness test to each item.

**Findings**

1. `MEDIUM` — Notes for the plan writer: bullets 1 and 3 are a list of code the writer read, which belongs in `relevant.md`.
   - Evidence: `buildSettings`, `recordingCommand`, `isNotice`, `composeReminder` and `reminderDue` are already listed in `relevant.md`, each with its why. The spec says "The list of code the writer read belongs here, never in the decision record."
   - Completeness test: what these bullets require ("D2 merges into the recording command", "D6 removes the notice and reminder branches") is already in D2 and D6.
   - Fix: cut bullets 1 and 3. Bullet 2 (the renderer must work per event and carry a subagent check and an exit convention) can stay as a hint, or move into D1 or D3. Bullet 4 (`Store.Interrupt` as the model for D7) can stay.

2. `LOW` — Constraints, the rewake-summary bullet carries how the conclusion was reached.
   - Evidence: the official-marketplace stdout path, the `@internal` entry field and the fallback order are research narrative. The plan writer needs only two rules: "a settings hook cannot show dynamic rewake text; the operator sees `Stop hook feedback`" and "set no `rewakeSummary`".
   - Fix: keep those two rules with the version and source. Move the internals to support-log Turn 2, which already summarises them.

3. `LOW` — Constraints, the JavaScript timer bullet ("above 2^31−1 ms fires at once") is said twice.
   - Evidence: the support log's Rejected alternatives already carries it. The decision it bounds, 86400 s, is fixed in D4 and justified in Auto-mode assumptions.
   - Fix: drop it from Constraints, or shrink it to a clause in D4 ("well under the JavaScript timer ceiling").

4. `LOW` — Open risks bullet 2 restates D5's rationale.
   - Evidence: D5's rationale already says that whether a waiter survives a turn or `/clear` is undocumented and that D5 covers it. Bullet 2 says it again and adds only "a lingering waiter holds one daemon connection".
   - Fix: fold that one cost into D4's Bound and drop the bullet.

5. `LOW` — Open risks bullet 1 is the one item that goes to the parent, but it is not phrased as a question.
   - Evidence: it describes the D4/D5 departure from the entry ("the pane path for posts goes", "the wake carries the message") in prose. The rubric routes parent questions as class `decision`.
   - Fix: lead with the question, for example "Accept a typed fixed bell (D5) for an idle strand with no live waiter, or drop it and let such mail wait for the next turn?", and cite D4/D5 instead of re-explaining them.

6. `LOW` — The Scope "In" list repeats D1–D10 nearly one for one.
   - Evidence: each "In" bullet is a one-line summary of a decision.
   - Fix: shorten "In" to a sentence that points at the decisions, and keep the "Out" list, which carries information no other section has.

**Other notes (not leanness)**

- `LOW` — D2's rationale says "Claude Code runs a settings event's hooks side by side", but Constraints cites no source for it, unlike the other Claude Code facts there. Add the hooks-docs citation (matching hooks run in parallel) next to the other Constraints facts.
- Auto-mode assumptions overlaps the support log's Question ledger. The spec requires both sections, so I am not flagging it.

**Tool calls:** none. My context already held both files, the spec and the rubric.
