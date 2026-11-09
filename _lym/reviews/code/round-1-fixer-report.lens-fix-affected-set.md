```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

The one finding I could not rule out is `internal/boardengine/doc.go:102`: its text still reads correctly, but it now covers the `runs` window only.

Task: fix-affected-set lens. I reran the search behind the findings whose premise is a set of sites: callers that form a title (F1) and old title wording (F5).

- **F1, callers of `TitleWithHandle`:** the only production caller left is `internal/orchcli/runs.go:56`, the `runs` window. The review excluded it by its own text, and the summary now says the same. `statusWindowTitle` was the only other production caller, and it now goes through `RecipeWindowTitle`.
- **F1, wording about the status window:** the only places that name `T4: tapestry` are `internal/loomcli/sharedbootstrap.go:380`, `internal/loomcli/doc.go:162` and `docs/overview.md`, and all are updated. `internal/loomcli/bootstrap_test.go:671,707` still show `T4: tapestry` and `T4: darn`. Those rows use a board with no shortname, so the titles are still correct.
- **F1, `internal/boardengine/doc.go:102`:** it says `TitleWithHandle` opens "a window's name with its run's handle, `T4: <name>`". It does not name the status window, so it is not wrong.
- **F5, stale wording:** no "slug opened with the handle", "bare slug" or "run's board handle" wording is left in the title code, tests or docs. The `bare slug` hits are about branch names and are unrelated.
- **F2, `terminalWindowTitle` callers:** the callers are `window.go` and the `windowTitle` seam in `cli.go`. The new tier 1 test covers the real function.
- **F3 and F4:** their sets are rows in the changed tables, and the new rows are in place. I did not search further.

Tool calls (3):
1. A grep for `TitleWithHandle` and the old wording across the Go and markdown files. My context had no repo-wide caller list.
2. A grep for `worktreeWindowTitle`, `RecipeWindowTitle`, `statusWindowTitle` and `statusStrandSpec` in non-test files, to confirm every call site of the changed signature. My context had no such list.
3. A grep for `terminalWindowTitle` and `windowTitle` in `internal/reedcli` outside `window.go`, `window_test.go` and the new integration test, to check for other callers. My context had no such list.
