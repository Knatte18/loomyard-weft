```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fixer's change set for stale siblings and unmatched twins (lens fix-consistency).

No findings of severity BLOCKING or MEDIUM. Two LOW findings:

1. **LOW**, `docs/overview.md:1846`: "The window opens with the run's board handle" is the twin of the line the fixer changed at `docs/overview.md:1059`. It was left as it was, so the two lines in the same doc now disagree.
   - The status window's name is now the console title, such as `ly: P3: T4: tapestry`. That title carries the shortname and the project handle as well as the run's own handle.
   - The changed text is the `statusWindowTitle` and `RecipeWindowTitle` change in commit `deb0d9d0d`.
   - Evidence: `docs/overview.md:1845-1846` and the rewritten line at `docs/overview.md:1059`.

2. **LOW**, `internal/boardengine/doc.go:104-109`: the package doc describes `IDEWindowTitle` and `TerminalWindowTitle` but not the new `RecipeWindowTitle`.
   - The doc lists the title functions and the form `<shortname>: <P#>: <T#>: <slug>`, but says nothing about the recipe standing in place of the slug.
   - Line 102 still reads as if `TitleWithHandle` names run windows generally.
   - It now serves only the `runs` strand, because the status window left it.
   - Evidence: a grep of `internal/boardengine/doc.go` and `docs/overview.md` finds no `RecipeWindowTitle`.

Checked and consistent:
- The `loomcli/doc.go` rewrite, `overview.md:1059` and the `summary.md` edit agree with each other.
- The `orchcli/runs.go` caller still uses `TitleWithHandle`, so the "runs keeps its name" decision holds.
- Nothing reads the status window's name back. `reedengine/windoworder.go` keys on `Display.Title` and the strand's `NameOverride`, not on window-name text. `reedengine/ownwindow.go:24` says a group window is found by its role, never by its window name.
- `TestStatusStrandAddSpec` and `TestStatusStrandSpec_...` keep passing, because their boards record no shortname.

Tool calls:
1. A grep for title consumers, `TitleWithHandle`, `Display.Title`, and tests and docs naming the status window. My context lacked the sibling and doc references.
2. Reads of `internal/boardengine/doc.go:96-110` and `docs/overview.md:1843-1848`, with a grep for `RecipeWindowTitle`. My context lacked those doc spans.
