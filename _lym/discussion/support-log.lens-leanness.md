```yaml
lens: leanness
outcome: reported
```

Task: leanness review of the decision record and support log. I looked for content to move or cut, applying the completeness-before-leanness test.

1. **LOW — `## Constraints`, the PATTERN bullets.** Several bullets only restate a PATTERN entry and add no constraint specific to this design. The plainest cases are `PATTERN-cli-cobra`/`PATTERN-cliwire-sole-wiring`, `PATTERN-run-process-owner` and `PATTERN-parent-lock`.
   - Evidence: each one repeats the entry's text and points at a decision that already cites it. For example, D13 already says "through `runproc`" and D12 already names `PATTERN-dobby-orch-pair`'s rule.
   - Fix: keep the bullets that state a design-specific binding: pull-request-mode, github-auth, remote-refs-lock, tracked-file-locks/durable-vs-ephemeral and shed-shared-machinery. Cut the rest. The plan writer gets the PATTERN list from its own prompt.

2. **LOW — `## Auto-mode assumptions`.** Each bullet restates a decision that is already stated in full: D6 ("no merge verb"), D3 ("refusals 1, 2, 4, 5"), D8 ("`on` no longer demands one") and D9 ("commit text from the description").
   - Evidence: this breaks the say-it-once rule, and the self-picks are already recorded in the support log's question ledger as Q2–Q5 and Q9.
   - Fix: shrink each bullet to a one-line pointer naming the assumption and its decision id, with no rule text. Keep the first bullet, the reading of "hub's rules for beam pull requests", because it is an interpretation rather than a restatement.

3. **LOW — the flock fact is stated twice.** The flock/close-on-exec fact appears as the first `## Constraints` bullet and again in the first `## Open risks` bullet.
   - Evidence: both say "the lock already frees on its holder's death".
   - Fix: keep the fact with its evidence in Constraints. Open risks should keep only the question for the parent.

4. **LOW — D1's last sentence is a doc/test edit list.** "The recipe's header comments, `pawlengine`'s and `landingshed`'s package docs, and the hold-boundary pin test follow the new rows" is a call-site/doc enumeration.
   - Evidence: the rubric's "Do not flag" section leaves enumeration to Plan.
   - Fix: cut it. The design does not depend on it.

5. **LOW — D2's Bound repeats D3.** D2's Bound restates D3's refusal 2 and the dobby mechanism, and D2's own third sentence already says the same thing ("covers a member filed in the window").
   - Fix: keep the Bound and drop the "it covers a member filed…" clause from the third sentence, or the other way round.

6. **LOW — the Rationale lines on D1, D10 and D14.** They mostly repeat the Goal: "the operator iterates the beam inside Beam-Await", "`lym beam ready` is the point of no return", "the orch drives the landing".
   - Fix: trim each to the one fact the Goal doesn't carry, or drop it. D13's and D9's rationales carry facts and should stay.

7. **LOW — the second `## Open risks` bullet (D9 in every recipe).** This is a known trade-off the writer chose (Q5). It is not a question only the parent can settle, so it doesn't belong in Open risks, which per the prompt feeds the parent's escalation.
   - Fix: move it into D9 as its stated Bound.

8. **Noticed, outside the leanness lens — D9.** D9 widens the change to every recipe's Finalize, beyond the entry's beam-only text. The board-scope lens should weigh it.

9. **Noticed, outside the leanness lens — the support log's "Turn 2".** It lists topics but no reasoning; the reasoning sits only in the ledger. That's thin rather than bloated, so it's no leanness finding.

Further tool calls: none. Everything I needed was already in my context.
