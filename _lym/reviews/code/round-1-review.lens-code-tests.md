```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, listing for each behaviour the diff adds or changes the test row that pins it.

All behaviours below are pinned by `TestRenderReadmeGolden` in `internal/boardengine/render_test.go`. I made no tool calls and judged from the diff already in context.

| Behaviour | Pinning row |
|---|---|
| README holds `## Prime` at level 2, with open projects nested under it at level 3 and deeper | The fourth row (`## Prime`, `### [P1 Ship it]`, `#### [P5 Later]`) |
| Prime's own entries come last, after its sub-projects | Fourth row (Tasks, Notes and Done of Prime follow P6) and the lone-beam row (`emptyProjectEntries(3)` last) |
| A running member shows under its project's Running | Fourth row (`##### Running (1)` holding T2 under Ship it) |
| Each project gets Tasks, Ready, layers, Draft, Notes by type and Done of its own | Fourth row (Ship it has Layer A, Notes `Other` and Done; Tidy up has Draft) |
| Member count in a project heading counts open members including sub-projects and excludes done entries | Fourth row (`Ship it (6)` with T4 done and excluded, P4 and P5 counted) and the sixth row (`(1)` for projects holding only a sub-project) |
| An open project under a done project counts as under the root | Fourth row (P6 Orphan sits at `###` under Prime) |
| An empty project renders Tasks, Ready (`_None._`) and Notes, with no flat member table or `_None._` member line | Lone-beam row and `emptyProjectEntries` |
| Project file shows its sub-projects as `##` sections, then its own entries at `##` | Fourth row's `project-ship.md` expectation |
| Heading levels stop at six | Sixth row (the chain of four projects, with the entries of `four` and `three` both at `######`) |
| Prio column is gone from project rows and stays on Notes tables | The golden header lines in the fourth row (`| # | Prio | Note |` kept, no project tables) |

Findings: none.

- **Not tested:** a project file whose project has sub-projects two or more levels deep. The depth cap and nesting are pinned only through `renderReadme`, base level 2. The file path uses the same `projectBody` and `projectSection`, so I rate this too thin to raise.
- **Test economy:** the sixth row covers the cap and no other row does. `emptyProjectEntries` is a plain string builder. The new rows duplicate no existing test.
- **Outside my lens:** `doc.go:75` still says "the Notes tables and the project sections" show the Prio column. The diff no longer renders project-section tables, so the affected-set lens should check it.

Tool calls: none.
