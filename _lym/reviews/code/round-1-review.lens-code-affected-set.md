```yaml
lens: code-affected-set
outcome: reported
```

**Task:** lens code-affected-set. For each change, I list what reads, calls or asserts it outside the touched lines.

**Per change, set found**
1. **Refusal "nothing depends on a project" removed** (`project.go`, `dependsOnProjectError`). Set: `doc.go` (updated), `project_test.go` (the old test is replaced by `TestProjectDependencies`), and the board skill and orch role docs. A repo-wide grep finds no leftover text outside `_lym`. `plugins/ly/skills/board/SKILL.md:19` says "only a task carries depends_on", which stays true. Covered.
2. **Dependency judgment moved into `Dependencies`** (`dependency.go`). Every non-test `DependsOn` reader under `internal/` is accounted for.
   - Converted readers: `dobbyreads.go:286` (`PendingDependencies`), `layer.go`, `render.go` `afterCell`, `siblings.go`, `store.go` `validateWrite`. Covered.
   - `PendingDependencies` consumers in `dobbyengine/starts.go:63,111` and `view.go:117` read the converted field. Covered.
   - Readers that never judge "met", so they need no change: `runlock.go:190-212`, `rename.go:93`, `replay.go:433,554`, `store.go:804` (the prune strip) and `render.go:800` (`renderDesigns`, which lists raw slugs by design). Covered.
3. **Cycle check follows a project to its members.** `store.go:353-384` (dangling, isolated and note checks) still run before it, so a project dependency passes them. The one asymmetry is that the isolate-guard at `store.go:410-418` only looks at a direct `depends_on`, which is unchanged behavior. Covered.
4. **README After, Ready and layer output.** `doc.go:203-212,236` is updated. The `render.go` file header is not: see the finding below.
5. **Siblings notice.** `doc.go` Siblings section and the `addedSiblings` comment are updated. Covered.
6. **`PATTERN.md` and `pattern/`.** The diff touches neither, and my context shows no declared PATTERN changes. Nothing to land. `PATTERN-refusal-way-forward` binds only the webster, shed and loom packages, so removing a board refusal is out of its scope. `PATTERN-test-speed` is respected: the new tests call `tiergate.Require(t, 0)` and `t.Parallel`.
7. **Commit messages.** Neither commit carries a `Pattern-Departure:` trailer, and none is needed.

**Findings**

1. **LOW · consistency** · `internal/boardengine/render.go:13`
   - The file header still says Ready and the layers add "the handles of the open entries each waits on".
   - The diff changed the After column's rule to "entries not done, a project judged by its landing members" in `doc.go:212` and the `afterCell` doc, but left this line.
   - Evidence: `render.go:13` against `render.go:590` and `doc.go:212`.

2. **LOW · consistency** · `internal/boardengine/doc.go:203-204`
   - The Ready paragraph says "A dependency on a done task does not count, and one on a running task does."
   - It then relies on the added line 236 for projects, and the generic "no open dependency" wording in `doc.go:203` is not reconciled with it. A project dependency that is met is also ignored, and that is not stated here.
   - This is a wording gap, not wrong behavior.
   - Evidence: `doc.go:203-204,236`.

**Tool calls (3)**
- Read `render.go:785-814`. My context lacked the `renderDesigns` body, to check whether it reads `DependsOn` as a judge.
- Read `render.go:1-30`. My context lacked the file header, to check whether it describes After and Ready.
- Grep of `doc.go` for the Ready, After and `isDone` wording, and of `internal` for `waitsOn` and `.members[` users. My context lacked a search for outside users of the new unexported helpers. They are confined to `boardengine`; the `websterengine` hits come from its own `members` fields.
