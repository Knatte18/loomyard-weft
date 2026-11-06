```yaml
lens: fix-consistency
outcome: reported
```

**Findings**

1. LOW. The `contracts/stencils/webster/webster-template-master.md` heading at line 236, "A plan conflict goes to the plan fixer first; a plan drift ends your run as stuck", is now false for `plan_order`.
   - Lines 240-242 send a `plan_order` refusal straight to `outcome: stuck`, so it never goes to the plan fixer.
   - Line 239 still says "a small plan fixer matches the cards to the tree" without limiting that to `plan_defect` and `tree_moved`.
   - Line 238 still lists `plan_order` under the same "that means" clause, so a reader skimming the heading gets the old routing.
   - The sibling `docs/overview.md:448` ("only a conflict whose fix would change what a card does ends the run stuck") still reads correctly, so it needs no edit.
2. No other findings.
   - The `fix-plan` mentions in `webster-template-plan-fix.md`, `llm-model-spec.md` and `docs/overview.md:645` raise no conflict.
   - The F2 rewording at line ~269 ("a `plan_conflict` refusal that names the way forward") agrees with the `plan_order` routing.
   - The F3 test edits match `planindex.CheckDeleteBeforeReference`, and `go vet ./internal/planglyph` was clean.

**Tool calls**
- grep over `contracts`, `specs` and `docs` for `plan_defect`, `tree_moved` and `fix-plan`, to find sibling mentions of the routing outside the edited lines.
- Read of `webster-template-master.md` lines 236-257 to see the edited section as it now stands, since my context held only the pre-edit text.
