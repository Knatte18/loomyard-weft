```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged each review proposal the fixer applied as proposed (F1, F3, F4, F5, F6, F7, F8, F9) for a concrete case where the proposal itself is wrong.

**BLOCKING**: decision-record.md, D10 as rewritten for F4, "In the family outcome dependents keep `depends_on` on the converted project", plus the matching D10 criterion and support-log ledger line.

- **Summary:** once every family member has landed, the converted project has no landing members and no done status, so an entry that depends on it never starts.
- **Case:** members land one at a time, and each landing prunes done entries. The support log's Turn 1 records that landing prunes. `Store.Prune` (`internal/boardengine/store.go:836-880`) removes every done entry. It keeps only a done project that a surviving entry names, never a done member task.
- After the last member lands and is pruned, the converted project is still open (D6 clears its status) and `Dependencies.members[project]` is empty.
- `metBelow` (`internal/boardengine/dependency.go:59-60`) returns `len(members) > 0 && ...`, which is false here. So `met` is false for good.
- Grep found no code that marks a plain project done when its members land. The only matches are docs and render rules over `internal/boardengine`, `landingdeps.go` and dobby.
- **Breakage:** the old D10 re-pointed dependents at the family's last parts. A pruned slug counts as met ("a slug the entries do not hold was pruned as done", `dependency.go:8`), so that design did let dependents start. The proposal makes the artifact wrong where it was right.
- **Criterion gap:** the new D10 criterion checks only that a dependent "does not start before every member is done". It never checks that the dependent starts afterwards, so it would pass with the stuck dependent.

**Other proposals checked, no finding:**
- **F1:** keeping labels unchanged is sound. A project needs a type label (`labels.go:46`), and the arras task already carries one.
- **F7:** the record step on a released part can be reached, since darn rows include `RecordWrite` (`contracts/recipes/darn-recipe.yaml:78`) and decision add checks `RecordWrite` (`internal/loomcli/decision.go:1040`).
- **F3, F5, F6, F8, F9:** I could not build a case against any of them.

**Tool calls:**
1. Grep for prune functions and project-membership doc lines in `internal/boardengine`: I needed to know whether done members are pruned, which my context lacked.
2. Grep of `darn-recipe.yaml` engines and the record-row checks in `decision.go`: I needed to know whether a filed darn part has a record row for F7's way forward.
3. Read of `store.go:825-880` (`Store.Prune`): I needed the prune survival rules for members and projects.
4. Grep for code that marks a plain project done after its members land: I needed to rule out a path that would close the converted project.
