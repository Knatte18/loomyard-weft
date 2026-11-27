```yaml
lens: code-scope
outcome: reported
```

Task: trace the diff's changes to the decisions that ask for them (code-scope lens).

**Per change → decision**

| Change | Traces to |
|---|---|
| View projection, `board-data.js`, revision functions, `Board.View`, `Outputs.Labels`, `unmetDependencies` and `migrateLoaded` refactors | D1, D3, D8 (card 1) |
| `Expecting`, `StaleEntryError`, check in the write section | D8 (card 2) |
| `board.html`, `ViewerPage`, `board.html` render output, replay claims, and the two page fix commits (keep an unlisted project; send an empty project) | D2, D3, D4, D10, D11 retry (card 3) |
| `boardserve` handler and writes, overview tree line | D9, D10, D13 (card 4) |
| `boardserve` run, state, launcher and watch loop, Windows DACL, `BoardServeDirName`, daemon-wakeups list and PATTERN background | D9, D11 (card 5) |
| `lock.AcquireWriteLockWithin` first try outside the deadline, plus its test row | forced by card 5: `run_test.go:118` takes free locks with a 10 ms `LockWait` |
| `open` and `serve`, openers, `spawnServer`, `RoleBoardServer`, `IsRunAgentSession`, `rerender` Short, doc.go updates | D5, D11, D12, D13 (card 6) |
| `board open` worktree task, two keys, docs | D6 (card 7) |
| Workspace-content seam, `BuildWorkspace` members, build-all-before-write | D6 and the seam entry (card 8) |
| `WorkspaceTasks`, ide hook, install in `main.go`, docs | D6 (card 9) |
| `testonlycode` allowlist adds and removals | forced; no net change over the range |

No change is unrequested scope as a whole. Finding 1 is a capability inside a traced change that goes past its decision.

**Per decision → changes that build it**

| Decision | Built by |
|---|---|
| D1 | projection |
| D2 | embedded page and render output |
| D3 | manifest and replay claims; the owner write goes through `Render` |
| D4 | `board.html` |
| D5 | `open` verb and openers |
| D6 | tasks, keys, seam, hook |
| D8 | revision and `Expecting` |
| D9 | server checks, state, launcher |
| D10 | routes, writes, page forms (gaps: findings 1 and 2) |
| D11 | lifecycle (deviation: finding 3) |
| D12 | run-agent refusal, empty strand name |
| D13 | package split, `runproc` role |
| Per-body README folds moot | nothing to build; none built |
| No `launch.json`, title premise stale | nothing built |
| Phone goal met by today's README | nothing built |
| Prime open chain and LY title stay out | nothing built |
| Seam keeps the workspace schema in `internal/vscode` | seam as written |
| `board.html` and `board-data.js` stay committed | D1 to D3 changes |
| `open`'s rerender refusal dropped (arbiter) | no refusal; the line is in the `file://` page |
| Parent confirmation of both | same |
| Opener waited on through `cmd.Wait` | `startOpener` |

**Findings**

1. **MEDIUM, design.** `internal/boardserve/write.go` (`opFields`, `applyWrite`, `create`, `upsert`).
   - `create` accepts `fields.kind` with any value, and neither `create` nor `upsert` refuses `"project"`. A `POST /api/write` with `{"op":"create","fields":{"kind":"project",…}}` therefore creates a project through `UpsertTaskReporting`. `edit` likewise has no kind check on its target, though a client would have to compute a project's revision itself, since the projection carries none.
   - D10 limits the writes to "create a task or a note" and "edit a task or a note", and Scope Out excludes "project create or edit".
   - Only the page's `<select>` limits the kind. No row in `server_test.go` mentions a project kind.
   - Evidence: `write.go:31` (`opCreate` fields include `kind`), `write.go` `create` and `upsert`, `siblings.go:148-165` (no kind guard), `project.go` beam refusal text (a plain upsert makes projects).

2. **MEDIUM, design.** `internal/boardengine/board.html` (`valuesOf` and the edit submit).
   - D10's "edit … the slugs it depends on" is only partly built. The edit form fills `depends_on` from `entry.waits_on`, which holds only the unmet dependencies.
   - When the operator changes that field, the submitted list replaces the stored `depends_on` and silently drops every met dependency. For example, a task depending on `[done-a, open-b]` shows only `open-b`. Adding `c` writes `[open-b, c]`.
   - Evidence: `board.html` `valuesOf` (`depends_on: entry.waits_on.map(slugOfHandle)`), `view.go` `viewEntry` and `render.go` `unmetDependencies`.

3. **BLOCKING, design.** `internal/boardserve/run.go` (`watchBoard`).
   - D11 and card 5 say the loop wakes on an `fswatch` event on `board.json`. The code calls `opts.Watch(opts.BoardPath)` with no names, so every entry changed in the board directory wakes it (renders, manifest and so on), although `Watch` takes a name filter.
   - Each event recomputes the revision at once with no one-second floor, which `PATTERN-daemon-wakeups` requires ("Any periodic wake is at least one second apart"; the summary says "wakes at most once a second"). A burst of events from one write's render causes back-to-back wakes.
   - The same diff words the PATTERN background and the `doc.go` as "the board directory", which follows the code rather than D11.
   - The record's carry-over block already lists `watch-loop-no-one-second-floor` (BLOCKING). It is still unfixed at HEAD.
   - Evidence: `run.go` `watchBoard` (`case _, open := <-events:` falls straight through to `BoardRevision`), `internal/fswatch` `Watch(dir, names...)`.

Outside my lens: the carried-over `board-html-not-pinned-to-lf` also still looks unfixed, since `git check-attr eol` reports `unspecified` for `board.html`.

**Tool calls**
1. `grep LockWait run_test.go`, `git log -1` for 581c3aff, and greps for `KindProject` and `UpsertTaskReporting`: my context lacked the test's lock wait, the commit body and whether an upsert guards project creation.
2. `sed` spans of `project.go` and `store.go`: my context lacked the project rules an upsert runs through.
