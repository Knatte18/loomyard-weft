# Card 3 — plan-lenses

**Edit:**
- `internal/burlerengine/template.yaml`
- `internal/burlerengine#TestConfigTemplate_DecodesAndIsSelfConsistent`

**Intent:**
Add the `branch-coverage` lens to the `twill-plan` fan and one sentence each to `forced-edits` and `criteria-coverage` (D7, D8, D9).

`template.yaml`:

- A new lens `branch-coverage`, in the fan's style and ending with the emphasis line: "Focus on branch coverage: list every branch and exit the cards' Intents introduce or change, whether or not the decision record names it (error returns, early returns, refusals, a loop's or scan's end without a match, a default case), and for each find the planned test row that reaches it. A branch no planned row reaches is a finding. Report anything else you notice too — emphasis, never exclusion."
- `twill-plan` lists `criteria-coverage`, `branch-coverage`, `row-reachability`, `forced-edits`, `test-economy`.
- `forced-edits` gains, before its emphasis line: "A new type, function or executor that must be registered in a table or list, such as a guard table, is a forced edit, and a registration no card lists is a finding."
- `criteria-coverage` gains, before its emphasis line: "Check that each path, comparison or default a card states uses the operand the record names, not a near one, such as the current directory's parent where the record names the worktree's parent."
- The plan rubric stays unchanged.

`TestConfigTemplate_DecodesAndIsSelfConsistent`: `wantLenses` goes from 22 to 23, the `twill-plan` entry count from 4 to 5, and the `twill-plan` order to the five names above.
It also asserts, over each lens text joined by `strings.Fields`, that `branch-coverage` contains "A branch no planned row reaches is a finding.", `forced-edits` contains "a registration no card lists is a finding" and `criteria-coverage` contains "uses the operand the record names, not a near one"; a dropped sentence or lens fails it.
The Plan-Write host reads the fan through `ResolveFan`, so `TestPlanTable`'s "template fan without burler.yaml" row, which wants `lensNames` of the resolved fan in order, puts `branch-coverage` in the rendered prompt with no edit.

**ImpactSummary:** Changes the embedded burler template's lens set and twill-plan fan only; a hub burler.yaml that overrides twill-plan keeps its own list, and only the burlerengine template test pins the counts.
