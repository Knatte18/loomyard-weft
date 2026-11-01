```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fixer's changes through the fix-consistency lens, looking for siblings, criteria and twins that no longer agree with the changed text.

1. **MEDIUM:** the F1 exception (front tasks excluded) was made in D4 but not in the places that say the same thing.
   - **Changed text:** D4's second hold bullet now holds released tasks only if they are "not a front task (an entry whose `front` field is true, D8)".
   - **Unchanged twins that still hold every released task:**
     - `PATTERN-adopt-hold` (PATTERN changes): "`lym adopt` holds it, and each released task not started under the subject".
     - Scope In, first bullet: "the released tasks not yet started under the adopted subjects".
     - Auto-mode assumptions: "A released task not yet started under an adopted subject waits for the check too (D4)".
     - support-log Q4: "plus released tasks not started under the subject".
   - **Why it matters:** the PATTERN entry is the binding rule, and as written it still covers the fix task. That is the deadlock F1 removed from D4.

2. **LOW:** "open" for a front task is never defined, and the new text relies on it in several places.
   - **Changed text:** D8 says "`fronttask` files no front task in a worktree where one stands open". D6 says "While any front task stands open in the subject's worktree … the check files none".
   - **Where the meaning comes from:** D8's start-hold bullet now ties the hold to the entry carrying "the empty status or a run status", and the D8 criterion says "a draft one lifts the start hold".
   - **The gap:** nothing says whether a draft front task (after a bare `lym batten cancel` by hand) still "stands open" and so blocks every later filing. This is the same wedge F3 named, now only reachable by a hand cancel.
   - `PATTERN-front-task` also still says "while it is open" without that definition.

3. **LOW:** D8's Rationale now undercounts the piece.
   - **Changed text:** D8's new bullets add a machine-local `fronttask` record and a `fronttask` removal.
   - **Stale sibling:** the Rationale, left unchanged, says "filing, the start hold and the land action are the three things both the adopt fix and a merge-in from parent need".
   - **The gap:** the piece now also removes front tasks and keeps a durable record, so the stated count is stale.

No further tool calls; my context already held everything this lens needed.
