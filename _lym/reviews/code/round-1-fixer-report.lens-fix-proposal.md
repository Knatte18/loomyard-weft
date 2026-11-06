```yaml
lens: fix-proposal
outcome: reported
```

**Findings**

1. **LOW** — `internal/websterengine/doc.go:585`, a sibling of the changed stencil text at `contracts/stencils/webster/webster-template-master.md:240-242` (F1).
   - **Summary:** `doc.go` still says Merriam runs `fix-plan` backgrounded for a `plan_conflict` refusal, with no `plan_order` exception.
   - **Evidence:** the line reads "Merriam runs `lym webster fix-plan` backgrounded, ends its turn, and on the verb's completion calls begin-batch again … on verdict `fixed`, and otherwise writes `outcome: stuck`".
   - **Why it is stale:** the stencil now sends `plan_order` straight to `outcome: stuck`. `doc.go:160` also defines `plan_order` as a cause of `plan_conflict`, so the two doc passages disagree for that cause.
   - **Not shown:** I did not read `doc.go:585`'s surrounding paragraph, so whether it is scoped to `plan_defect` and `tree_moved` is unconfirmed.
   - **Severity:** it fits the rubric's LOW (a sibling left stale), because the doc sentence only describes the common path.

2. **LOW** — `contracts/stencils/webster/webster-template-plan-fix.md:1` and `:9`, a twin of the F1 change.
   - **Summary:** the plan-fixer stencil header still says it is spawned "after begin-batch refuses with plan_conflict", and the comment is not narrowed.
   - **Evidence:** the grep output shows line 1 with that wording.
   - **Why it matters:** `fix-plan` is now never run for the `plan_order` cause under the stencil's routing, so the comment is slightly broader than the behaviour.
   - **Not shown:** I did not read line 9 beyond its first words.

**Challenges that found no breakage**

- **F1 (`plan_order` skips `fix-plan`):** I built the case where the plan fixer could in fact fix a `plan_order` refusal. The plan-fix stencil's own mechanical/plan-writer split, as the reviewer described it, sends any card-reshaping finding to the plan writer, so a move of a delete is not mechanical. I found nothing contradicting that, so the skip holds.
- **F1 wording:** the edited paragraph now says "skips the rung below" and "For `plan_defect` and `tree_moved`:", which scopes numbered steps 1-3 correctly. The sentence at line 238 still lists `plan_order` as a possible cause, which is consistent.
- **F2 (stop quoting the way-forward text):** the replacement text says a `plan_conflict` refusal "names the way forward". That is true for both rendered variants (`plan_edit` for the order case and the rebaseline wording otherwise). I found no state where it is wrong.
- **F3 (use `planindex.CheckDeleteBeforeReference` in the test readers):** the constant is a plain string const, and `Finding.Check` already compares against it in `deleteorder.go`. The fixer's `go vet ./internal/planglyph` ran clean, and no import cycle was reported (`planglyph/index.go` already imports `planindex`).

**Tool calls**

1. `grep -rn "plan_order\|fix-plan"` over the Go, md and yaml files outside `_lym` and `.lym`: to find sibling text that references the plan-conflict routing, which my context lacked.
