```yaml
lens: fix-way-forward
outcome: reported
```

fix-way-forward lens: checking that each refusal, halt, next step and criterion the fix added can be reached and leads somewhere.

**List: the ways forward the fix introduced**
1. D8: a held refusal in a round at or past the hard cap halts the segment at its budget, with a Reason naming the refusal's cause and `lyx loom resume`, or "the operator's own halt handling".
2. D8: below the hard cap, a held refusal retires the decision and re-escalates with cause `decision`; this is unchanged apart from the new hard-cap limit.
3. D8 brief: a `continue` runs another round, and a listed BLOCKING finding reaches that round's fixer only if the new reviewers raise it again.
4. New criterion: a hard-cap accept over a held refusal halts with a Reason naming the cause and `lyx loom resume`, and does not re-escalate.
5. Amended criterion: a `continue` after re-escalation runs another round with an empty focus file.

**Findings**

- **MEDIUM**: D8's hard-cap sentence "a held refusal halts the segment at its budget, as a refused judgment does today, with a Reason naming the refusal's cause and the way forward: `lyx loom resume`…", and the new criterion "halts the segment with a Reason naming the refusal's cause and `lyx loom resume`".
  Summary: the halt "as a refused judgment does today" cannot carry the refusal's cause, because a budget halt discards the producer's Reason.
  Evidence: `internal/shedengine/run.go:415` builds the persisted reason as `"%s for %s; way forward: %s", ReasonBounceBudgetExhausted, def.Name, s.budgetWayForward()`.
  `internal/shedengine/doc.go:29` says the budget-exhaustion reason is used "regardless of the producer's Reason".
  The way forward is the shed-wide `loomBudgetWayForward` (`internal/loomcli/wiring.go:297`).
  So the cause in `rejudge`'s degrade Reason never reaches the persisted stop, and the criterion cannot be met through the budget-halt path.
  A Reason that carries the cause needs a different halt, such as a Stuck with `OutputPointer.Halt` (`internal/shedengine/doc.go:98`: it "blocks the run at its own row with the producer's Reason … and is budget-exempt").
  Alternatively, the text drops the claim that the Reason names the cause and relies on the loom budget way forward.
  Whichever is chosen, the record must say how a resume then re-judges.
  A resume of a budget halt "grants exactly one more budget" (`doc.go:50`), and a second refusal spends that unit and halts again.

- **LOW**: D8's hard-cap sentence, "or the operator's own halt handling".
  Summary: this alternative way forward names no command or step, so it cannot be carried out as written.
  The other alternative, `lyx loom resume`, works only if a fresh judge rules CONTINUE or CIRCLING.
  The judge stencil makes that likely, since it says "Never relabel a BLOCKING finding downward" (`contracts/stencils/bouncer/bouncer-template-judge.md:60`).
  If the judge rules CONVERGED again, each resume buys one unit and halts again, and nothing else is named.
  Evidence: the quoted text names no verb.
  The halt is otherwise bounded by the operator, so this is LOW.

No finding on items 2, 3 and 5:
- **Item 2:** the hard-cap limit removes the loop F2 described.
- **Item 3:** this matches the perch order Bouncer → Burler-Review → Burler-Fix in `contracts/recipes/loom-recipe.yaml`, and the empty focus after CONVERGED (`approved_focus_lists`: "an empty `focus`").
- **Item 5:** this follows from the same.

Tool calls:
1. Grepped `internal/loomcli` resume and `internal/shedengine/doc.go` for budget, to learn what a resume grants after a budget halt; the context did not show resume semantics.
2. Located the judge stencil, to check whether a re-judge learns of the refusal; the context had not read the stencil.
3. Grepped the judge stencil for CONVERGED/BLOCKING, to see whether a fresh judge would rule differently.
4. Read the judge stencil's decision-rule lines 50–70, for the rule against relabeling BLOCKING downward.
5. Grepped `ReasonBounceBudgetExhausted` and `BudgetWayForward`, to see how a budget halt composes its persisted Reason; the context lacked `run.go`.
