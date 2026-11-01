```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff (recipeCell, entryCell, constant, doc.go).

Per-function list:

- `recipeCell`, Recipe "" and Profile "": returns `tapestry`. Correct.
- `recipeCell`, Recipe "" and Profile "light": returns `tapestry · light`. Correct, and matches the Idea's `tapestry · light`.
- `recipeCell`, Recipe "darn" and Profile "docs": returns `darn · docs`. Correct, and matches the updated golden row for T3.
- `recipeCell`, Recipe "darn" and Profile "": returns `darn`. Correct.
- `recipeCell`, Profile "a|b": `cellText` escapes it to `a\|b`, so the row stays one table row. Correct.
- `recipeCell`, Profile " " (whitespace only): `t.Profile != ""` is true, so `cellText` yields "" and the cell reads `tapestry · ` with a trailing separator. This is not a defect. The profile name comes from a validated run-profile name, not from untrusted input, and the code-quality rules say to trust the caller.
- `recipeCell`, Recipe "" with `defaultRecipe` used directly: `defaultRecipe` is a constant that never needs escaping. Correct.
- `entryCell`, with the profile bullet removed: title, draft prefix, brief bullet and settings bullets are unchanged. A task with a profile and no settings now renders only the title and brief. Correct.
- `entryCell`, Notes and Done paths (render.go:527, 553): these call `entryCell` without a Recipe column. Notes carry no profile, so dropping the profile bullet loses nothing. A Done task that has a profile no longer shows it anywhere, because the Done table has no Recipe column. This is consistent with the Idea, which only addresses the Recipe column, and `doc.go` states that Done has none. It is a scope observation, not a defect.
- `recipeProfileSeparator` constant: `" · "` is a multi-byte UTF-8 string inside a markdown table cell, so no escaping is needed. Correct.
- Call sites (render.go:471 for Ready, layers and Draft; render.go:493 for Running): all four task tables go through `recipeCell`, so the profile reaches every one of them. Correct.
- The grep for `profile: ` found no other renderer or consumer of the old bullet. Correct.

Findings: none.

Tool calls (3, all in the shared context before this fork, none by me):
1. Not needed beyond the manifest. I made no tool calls of my own, because the manifest and the inherited reads already held the diff, the call sites at render.go:471 and 493, `cellText`, and the grep results.
