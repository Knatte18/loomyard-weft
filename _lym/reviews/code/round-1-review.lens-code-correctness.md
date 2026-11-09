```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens on the title diff. I ran one concrete input through each changed function and branch.

**Per-function list**

- `consoletitle.Title`:
  - Input `("LY","","","prime")` gives `ly: prime`. Correct.
  - Input `("","P3","T7","slug")` gives `P3: T7: slug`. Correct.
  - Input with every part empty gives `""`. Harmless, since callers always pass a slug.
- `consoletitle.Workspace`:
  - Input `("LY","loomyard")` gives `ly: loomyard workspace`. Correct.
  - Input with an empty shortname gives `bm workspace`, as before. Correct.
- `workspaceWindowTitle` (hubworkspace.go): `ReadShortname` returns `found=false` and `""`, so the title is `<name> workspace`, as before. Lower-casing now moves into `consoletitle`. Correct.
- `worktreeWindowTitle`, prime branch (`slug == primeName`):
  - Input `("main","main")` gives `ly: prime` in VS Code and `ly: orch` in a terminal. It never reads the board. Correct.
  - Input with `primeName == ""` skips the branch, so the prime is titled `ly: <slug>`. This is the documented degrade.
- `worktreeWindowTitle`, `ReadTasks` error branch:
  - Input is an unreadable board with slug `bm`, a beam. The result is `ly: bm`, not `ly: beam` or `ly: orch`.
  - The `ReadTasks` error branch returns before the beam test, so a beam loses its label and not only its handles.
  - The doc comments say only "leaves out every handle", so this deviates mildly from them.
  - I judge it LOW, `design`.
  - A board that is absent, not corrupt, reads as an empty list, so a beam is titled by its slug there too. That follows from the entry not being on the board.
- `worktreeWindowTitle`, beam branch (`KindProject && Beam`):
  - Input `bm` gives `P2: beam` or `P2: orch`. Correct.
  - A done beam still gets its own handle and label. That is correct, since the beam is its own `<P#>`.
- `worktreeWindowTitle`, task branch:
  - A task in project P3 gives `P3: T7`. Correct.
  - A task with no project gives `T12`. `bySlug[""]` is the zero `Task` (a real entry with an empty slug cannot exist), so the project handle is empty. Correct.
  - A task under an open beam gives the beam's handle through `ParentBeam`.
  - A task in project P6 under open beam P2 takes P2. This matches the Goal and the `deep` test row.
  - A task whose beam is done: `ParentBeam` skips the done beam and returns `""`. Then `entry.Project` is the done beam's slug, and the title uses that beam's handle. Right.
  - A task whose `project` slug is missing from the board: `bySlug[project]` is the zero `Task`, so `<P#>` is left out. Right.
  - An entry that is not on the board gives `ly: stray`. Right.
  - A cyclic project chain makes `ParentBeam` return `""`, and the code falls back to `entry.Project`. No panic.
- `IDEWindowTitle` and `TerminalWindowTitle`: they differ only in labels. The VS Code prime reads `prime`, and a terminal on the prime's session reads `orch`. This matches the Goal table and the summary.
- `battencli.commandDeps` `Title`:
  - It passes `location.WorktreeName` as `primeName`. That is the prime only because batten commands run in the prime. If `location` were a beam, no slug would equal it, so the title would still be a task title. Correct either way.
  - With no shortname and no handles it returns the bare slug, which the updated test rows pin.
- `reedcli.terminalWindowTitle`:
  - `ResolveWorktree` fails (missing directory, pruned worktree, not a git repo) and it returns `filepath.Base(dir)`, as the old default did. Correct.
  - `ResolveWorktree` takes the walk-up worktree root from `dir`, so a subdirectory of a worktree gets the worktree's name. That is better than the base name of the subdirectory.
  - `PrimeName` fails: it logs a Warn, and `primeName == ""`, so the prime's own window reads `ly: <primeslug>` and not `ly: orch`. This matches the doc comment. It is a mild mistitle on a git failure, within what the diff states.
- `windowCmd` default-title branch: `--title ""` still counts as unset, as before. An explicit non-empty title wins. Correct.
- `ideengine.lookFor`: the title now comes from `IDEWindowTitle` before the prime and beam branches. The prime gives `tst: prime`, a beam `tst: P1: beam`, and `t1` gives `tst: P1: T1: t1`. The colour logic is untouched. The title and `beamColors` each read the board, so two reads can differ only by a concurrent write, with no effect on correctness. `strings.` is still used twice in `spawn.go`, so the import removal is not an issue.

**Findings**

1. LOW, `design`, `internal/boardengine/handles.go:196-199` (the `ReadTasks` error return in `worktreeWindowTitle`): an unreadable board titles a beam as `<shortname>: <slug>`, not `<shortname>: beam` or `<shortname>: orch`. The label is lost with the handles, though the doc comments promise only the handles are left out. `AnUnreadableBoardLeavesOutTheHandles` pins only a task row. Evidence: `handles.go:196-199`, `handles_test.go` row `AnUnreadableBoardLeavesOutTheHandles`.

No other defect found.

**Tool calls**
1. `grep -c 'strings\.' internal/ideengine/spawn.go` plus a read of `lymcwd.resolveCore`: my context lacked the import usage after the removal and the walk-up behaviour of `ResolveWorktree`.
