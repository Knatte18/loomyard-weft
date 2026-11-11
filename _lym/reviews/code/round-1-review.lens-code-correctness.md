```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. For each changed function or branch I ran one concrete input.

**Per-function list**

- `consoletitle.Title`, all parts set (`T2`, `LY`, `P1`, `adopt-worker-record`): `T2 [ly:P1] adopt-worker-record`. Correct.
- `consoletitle.Title`, shortname and project both empty, lead `T7`: the brackets are dropped and the result is `T7 slug`. Correct.
- `consoletitle.Title`, empty lead and empty project: `[ly] stray`. Correct.
- `consoletitle.Title`, only a name: `tapestry`. Correct.
- `consoletitle.Title`, empty name with a lead (`ORCH`, `ly`, `P0`, ""): the result is `ORCH [ly:P0] ` with a trailing space. The name is always appended, even when empty. No caller passes an empty name today: the name is the slug or the recipe, and an empty slug would already be broken upstream. This is a boundary NIT at most (see below).
- `consoletitle.Workspace`: `[ly] loomyard workspace`, and `bm workspace` when the shortname is empty. Correct.
- `worktreeWindowTitle`, prime branch (`slug == primeName`, non-empty `primeName`): `ORCH [ly:P0] root`, with or without a readable board. This is correct, since the check runs before `ReadTasks`.
- `worktreeWindowTitle`, empty `primeName` (`RecipeWindowTitle`, or a caller that cannot resolve the prime): the prime branch is skipped and the slug is treated as an ordinary worktree, giving `[ly] main` when it is off the board. Correct.
- `worktreeWindowTitle`, board read fails: `Title("", shortname, "", name)` gives `[ly] gate-shard`, and a beam gets no `ORCH` lead (`[ly] bm`). The warning is logged and nothing is swallowed. This matches the doc ("titles every worktree but the prime by its slug alone") and the tests.
- `worktreeWindowTitle`, slug not on the board: `[ly] stray`. Correct. The new `onBoard` guard stops an off-board worktree from being titled as a root task.
- `worktreeWindowTitle`, beam entry (`Kind==KindProject && Beam`): `Title(orchLead, shortname, entry.Handle, name)` gives `ORCH [ly:P2] bm` from `ConsoleWindowTitle`. From `RecipeWindowTitle` (`orchLead ""`) it gives `[ly:P2] pawl`. Correct. A beam with an empty handle degrades to `ORCH [ly] bm`, which is acceptable.
- `worktreeWindowTitle`, task directly under root (`ParentBeam` returns "" and `entry.Project` is ""): `projectHandle` stays `rootProject.Handle`, giving `T12 [ly:P0] adopt`. Correct.
- `worktreeWindowTitle`, task whose `Project` names an entry missing from the board: `bySlug[project].Handle` is "", giving `T10 [ly] orphan` (see `ParentBeam` at `parentbeam.go:24-27`, which breaks on the miss). This follows the change description's stated design. The alternative reading, "chain ends at root takes P0", could be argued, but it is a design choice and not a defect.
- `worktreeWindowTitle`, task under a done beam: `ParentBeam` skips it and `entry.Project` is used, so the handle is the beam's `P8`. Correct and covered by a test row.
- `worktreeWindowTitle`, entry with an empty `Handle`, task or not: the lead is empty and the result is `[ly:P0] slug`. Correct.
- `worktreeWindowTitle`, project-loop cycle: `ParentBeam` returns "", so `entry.Project` is used and `bySlug[project].Handle` is read. No panic and no wrong state.
- `ConsoleWindowTitle` and `RecipeWindowTitle`: thin wrappers that pass the right `orchLead`. The recipe name replaces the slug only in the name position. Correct.
- Callers (`battencli/wire.go`, `ideengine/spawn.go`, `reedcli/window.go`) only swap the function name. Correct.

**Findings**

- NIT, design, `internal/consoletitle/consoletitle.go:371`. `Title` appends `name` unconditionally, so an empty name leaves a dangling trailing separator (`ORCH [ly:P0] `). The doc says "an empty part is left out with its separator", which this case contradicts. It cannot happen with current callers.

No other defects found.

**Tool calls**

1. `grep -n -B6 -A22 '^func ParentBeam' internal/boardengine/*.go` — read `ParentBeam`'s behaviour for missing-project, cycle and done-beam chains; my context held only its call site in the diff.
