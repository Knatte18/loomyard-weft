```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff to `raddleModeRules` in `internal/loomengine/raddletable.go`.

**List (function or branch, case run, correct or not)**

- `raddleModeRules`, `RaddleModeInteractive` branch, with proposalPath="" and slug="": the string is built purely from literals and ignores both arguments, so nothing can fail. Concatenation is well-formed: every segment ends with a trailing space except the last, and the added segments (`raddletable.go:55-57`) keep that shape. Correct.
- Same branch, rendered text: the sentence at lines 55-56 asks for a recipe column (such as `darn`), a profile column (such as `solid`) and a dependencies column by part slug. This matches the Idea. Correct.
- Same branch, the new sentence at line 57 ("give only the reasons ... never the recipe or profile again"): it does not contradict line 60, which tells the agent to revise the recipes and profiles with the operator. The ban applies to the prose around the table, and the revision is an edit to the file. Correct.
- `raddleModeRules`, `RaddleModePerch` and default branches: the diff does not touch them. Correct.
- Template substitution: the text sits inside the `{{.mode_rules}}` value. The new backticked `darn` and `solid` are a value, not template source, so they are not re-parsed. I did not run the renderer to confirm this, which makes it an inference. It matches how the existing `` `lym loom raddle-propose` `` backticks already pass through. Correct.
- The `raddleModeRules` doc comment (`raddletable.go:41-48`): it matches the rendered interactive text, and the lines follow in order. Correct. Line breaks are not a finding under the rubric.

No findings from this lens.

Tool calls: none. The context already held the full diff and `raddletable.go`.
