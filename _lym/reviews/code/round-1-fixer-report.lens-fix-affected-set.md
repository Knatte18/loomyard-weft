```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set lens. I reran the set searches behind F1/F3 (RC condition wording), F4 (readers of `@lym_strand_name`), and F5/F6 (stale RC and `--dir` wording).

Findings, one entry per search:

- No findings (F1/F3). Every Go and doc mention of when the RC button is drawn now names `@lym_strand_name` and `@lym_button_dir`: the `bar.go` header, the `bar.go` condition comment, and `doc.go:212`.
- LOW (F1). `internal/reedengine/bindings.go:212` still reads "the RC button is drawn only while it [the click line] is set".
  - The sentence stays true as a necessary condition, but it is now incomplete next to the new three-part condition.
  - It is the one sibling statement of the draw condition the fix did not update.
- No findings (F4). The `@lym_strand_name` readers are `buttonclick.go:248` and `remotecontrol.go:37`, and the docs name both. `doc.go:170` and `doc.go:175` already covered the RC click, and `paneoptions.go` now matches.
- No findings (F5). The old wording ("orch session's status bar", "orch strand's role") is gone from every non-test file. `orchcli/doc.go:290` and `orchengine/doc.go:116` carry no stale text.
- No findings (F6). The old `--dir` help text "run directory of the session" is gone from every non-test file.

Tool calls:
1. Grepped RC-button draw-condition wording across non-test `.go`, `.md` and `.yaml` files. My context had only the files the fixer edited, not the rest of the tree.
2. Grepped `strandNamePaneOption` and `@lym_strand_name` across non-test `.go` and `.md` files to list every reader and doc. The review named only the paneoptions sites.
3. Grepped the F5/F6 stale phrases and `remote_control_name` across non-test files. This checks that no sibling doc kept the old wording.
