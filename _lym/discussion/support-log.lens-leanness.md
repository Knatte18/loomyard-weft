```yaml
lens: leanness
outcome: reported
```

Task: a leanness review of the decision record and support log, looking for content to move or cut.

Findings:

1. LOW — Goal. The paragraph that maps the five questions to D-ids repeats D13 and the decision headings. Evidence: Goal lists "a linked-only change (D1), a failed build (D5)…", and each decision already answers its question. Cut it down to the one-sentence goal.

2. MEDIUM — Constraints, "Today's code, read for this record". This bullet describes today's behaviour, and the Decisions already state each fact where it matters: D5 covers the failure-only map, D12 covers `Unbuilt`, D9 covers the slot held for the whole call, D11 covers the anchor wait, D4 covers spawn-marked covers. The bullet adds no constraint. Move it to the support log's Turn 1. The completeness check passes, because every fact it holds already sits in a decision's rationale.

3. LOW — Constraints, the `slots: 3` bullet. This duplicates D9's Bound and rationale. It could be folded into D9 as one clause. Keep it only if Plan needs the hub's value, and D9's Bound already gives the rule.

4. MEDIUM — Auto-mode assumptions. Four of the six bullets (function grain, no quiet-pool wait, prime worktree, no age backstop) restate D3, D9, D12 and D7 as settled decisions, so they are not assumptions. Two items are real assumptions and should stay: "no measurement runs before the design" and "spawn-marked tests never measured, hand calls included". The second contradicts nothing, but D4 should say it. Cut the four.

5. LOW — Notes for the plan writer, first bullet. The file pointers duplicate `relevant.md`, which is Plan's code head start under the spec. Cut that bullet and keep the build-split hint, which itself repeats D13's last paragraph. Cut one of the two.

6. MEDIUM — D13's last paragraph, the build order split. Ordering build parts is raddle's split work, so it is plan-level sequencing in a Decision. It carries one requirement: the selector-dependent parts follow `coverage-test-select`. Keep that dependency as one sentence and drop the list of parts that need no selector, or move it to Notes. Don't keep it in both places.

7. LOW — D1's last paragraph ("So a change in a linked-only package…"). It re-derives the three outcomes just listed in D1. The only new content is the interim-rule sentence about `testmap-cross-package`'s criterion, and D13 says that again. Keep one of the two.

8. LOW — Review criterion under Acceptance. "D13's changes appear on the stored records… and the note… is on the board" checks parent actions after the run, not text the diff leaves. Code review can't check it from a diff. Consider dropping it or rephrasing so it checks only the record text. This is outside a pure leanness view, but worth a look.

9. LOW — Support log, Turn 3. It repeats the question ledger's Q3, Q5, Q6 and Q8 one for one. Since the support log is not read by Plan, this is acceptable, so I'm noting it only.

No further tool calls were made.
