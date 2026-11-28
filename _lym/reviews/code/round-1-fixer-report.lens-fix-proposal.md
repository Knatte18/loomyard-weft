```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the review proposals the fixer carried out (F1–F4, lens fix-proposal).

**LOW: the new bullet in `burler-review-format.md` ("A finding whose `fix` carries another reviewer's change it may not reject…", F3)**
The bullet does not say who may not reject, and it does not mark itself as an exception to the rule above it.
- Evidence for the actor: in "another reviewer's change it may not reject", the only antecedent of "it" is "A finding". A finding rejects nothing; the actor is the reviewer, whom this file addresses as "you".
- Concrete case: a kept parent finding's change rests on a fact the code disproves. Two bullets up, the file says "**Never** write a `fix` that rests on a fact the code disproves", and to raise class `decision` when no fix holds. The new bullet says to keep that change and append the disproved fact, but never says it overrides the "Never" or the reclassification. `burler-parent-consolidation.md` also fixes that finding's class to the parent item's class, or `design`.
- A change that would settle it: "As the one exception, a `fix` carrying a change you may not reject, such as a kept parent finding's, keeps that change and its class, and appends …".

I built no case against the other three proposals:
- **F1:** removing the first-caller rule matches `PATTERN-test-only-code` ("A card may therefore add a function that only a later card calls").
- **F2:** the ordered dispositions cover a disproved premise, a disproved fix-only fact and the write-surface deferral.
- **F4:** a two-choice `fix` on a carried finding still has the fixer's route of deferring a tradeoff to the parent.

Tool calls made: none.
