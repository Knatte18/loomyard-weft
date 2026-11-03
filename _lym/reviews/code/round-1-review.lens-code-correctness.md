```yaml
lens: code-correctness
outcome: reported
```

Task restated: check the correctness of every function and branch the diff changes (lens: code-correctness).

**Per function, the case I ran:**
- `TitleWithHandle`, board present with a handle: `T4: name`. Correct.
- `TitleWithHandle`, board file and legacy files both absent: `Store.Load` returns no error and an empty list (`store.go:195-244`), so it returns `name` and logs no Warn. Correct, and no Warn fires on a hub with no board.
- `TitleWithHandle`, unreadable or wrong-version board: it returns the error, the function logs Warn and returns `name`. Correct.
- `TitleWithHandle`, side effects: `Load` never writes. `NewStore` passes a nil vocabulary, so a note's letter is undecided and its stored handle is kept (`handles.go:65-83,112-115`). A task or project without a stored handle gets one in memory only, which reads as the handle the board would assign. Correct.
- `TitleWithHandle`, slug match: it matches `task.Slug == slug`, and done entries are included. Correct.
- `statusWindowTitle`: `location.WorktreeName` is the worktree directory name, which equals the entry slug in a task worktree and a beam worktree. Correct. It also titles `P1: pawl` for a pawl run in a beam worktree, which is acceptable.
- `runsWindowTitleIn`, prime: the board is not read and the result is `beam: prime`. Correct.
- `runsWindowTitleIn`, beam with no handle or an unreadable board: `beam: <slug>`. Correct.
- `ensureRunsStrand`, `c.runsWindowTitle()`: the call is made before the keep check, so the board is read on every bring-up even when the strand is kept, which only costs a read. It is never nil in production, because `wireCLI` sets it. A hand-built `orchCLI` with a nil `runsWindowTitle` would panic, but `start_test.go:168` sets it.
- `CommandProducer.Call`, `{title}`: it runs only when `{title}` is present, and the value goes through the same quoting as the other values. A hub-layer `{slug}` command is untouched. A nil `deps.Title` with `{title}` in the command would panic, and `envkit.FullEnv` sets it. Loom's `intentionallyNil` entry covers the loom wiring. Correct.
- `commandDeps.Title`, slug: `slug` is the task slug, and the board dir is the hub's. Correct.

**Findings**

1. **LOW, design.** `internal/loomcli/sharedbootstrap.go:294-301` and `internal/orchcli/runs.go:92-93`.
   - Both `ensureStatusStrand` and `ensureRunsStrand` keep a live, current-build strand without comparing its title.
   - A handle assigned after the window opened never reaches the title. A handle that moves on a promote or type change is not corrected either, because the strand is only replaced when it is dead or the build changed.
   - The title is correct only for a window opened after the handle exists, e.g. a beam brought up before the project got its handle, or a status strand left running across a promote.
   - The entry says "read when the window opens", so this is defensible. It is still worth deciding whether a title mismatch should count as a reason to replace. The fix would need the existing title, which `StrandStatus` may not expose.

No other defects found in my lens.

**Tool calls**
1. Read `store.go:210-260`: the rest of `Load`, to check for side effects and missing-file behaviour.
2. Read `handles.go:40-130`: `handleLetter` and `assignHandles`, to check the nil-vocabulary behaviour.
3. Grep `func NewStore` in `store.go`: to confirm the vocabulary is nil.
