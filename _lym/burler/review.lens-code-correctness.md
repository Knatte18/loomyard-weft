```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff's functions, branches and error paths (read-only).

**Cases run**

| Function or branch | Case run | Result |
|---|---|---|
| `cli.go` `openCmd` / `viewerHub` | run agent; `--board-path`; strand set or empty for `OpenerCap` | correct |
| `cli.go` `serveCmd` | config load, resolve, register, listen or state write fails in the spawned child | **wrong** (F3) |
| `runOpen` | lock free, held, stale state, spawn error, opener error, cap | correct, bar a narrow exit race (F8) |
| `spawnServer`, `startOpener` | start failure, log append, detach and reap | correct, apart from F3 |
| `boardserve.Run` | lock held, leftover state, exit and drain order | correct |
| `Run`, `Serve` goroutine | accept error | **error lost** (F7) |
| `Run` under a manual `serve` | `Log` is the terminal | **log left empty** (F9) |
| `watchBoard` | burst of events, closed channel, read error, idle | **no floor on events** (F4) |
| `Live`, `ReadState`, `WriteLauncher`, `Probe`, `hardenFile` | missing directory, unknown key, temp file mode (`CreateTemp`, 0600) | correct |
| `ServeHTTP`, `serveAPI`, `serveBoard`, `serveFile`, `serveWrite` | wrong host or token, `after=""`, drain, encoded separator, oversized body | correct |
| `decodeWriteRequest`, `applyWrite`, `create`, `upsert`, `setStatus` | `null` body, unknown field, empty revision, project slug, each status | correct |
| `server.promote` | Sketch whose body changed after its record | **warning dropped** (F6) |
| `buildView`, `viewEntry`, `usedSections`, `View`, revisions | empty board, no `board.json`, done project, nil labels | correct |
| `Expecting`, `checkExpected`, `boardCriticalSection` | removed, changed, existing, new | correct |
| `renderFiles`, `RenderToDisk`, `unmetDependencies`, `Claims` | missing decision file, met dependency | correct |
| `AcquireWriteLockWithin` | wait 0, free and held | correct |
| `workspaceMembers`, `regenerateHubWorkspace`, `BuildWorkspace`, `markBeamWorkspace`, `WorkspaceTasks`, `WorkspaceContent` | no hook, refused name, unreadable anchor, `LymBinary` error | correct |
| `writeWorktreeTasks`, keybinding entries | empty `lymPath` | correct |
| page `valuesOf` and edit submit | task with one met and one unmet dependency | **met dependency deleted** (F1) |
| page `fill` / `isDirty` | body containing `\r\n` | **permanently dirty** (F2) |
| page `panelFor` on a kind change | unsent note edit, entry becomes a task | **edit discarded** (F5) |
| page action button with a dirty form | another writer changed the title first | **overwrite not refused** (F10) |
| page `markdown`, `safeTarget`, `request`, `follow`, `render` | hostile targets, empty revision, removed entry | correct |

**Findings**

**F1. BLOCKING, design: saving a task's dependencies deletes its met dependencies.**
- Location: `internal/boardengine/board.html`, `valuesOf` and the edit submit.
- The `depends_on` field is prefilled from `entry.waits_on`, which holds only unmet dependencies (`view.go` `viewEntry`, via `unmetDependencies`).
- Case: a task depends on `a` (done) and `b`. The field shows `b`. The operator adds `c` and saves. The page sends `["b","c"]`, and the upsert replaces the list, so `a` is gone with no message.
- A `waits_on` handle that matches nothing in `board.entries` or `board.projects` is also sent back as a slug.
- The projection carries no full dependency list for the form to use.

**F2. MEDIUM, design: a field the browser normalises reads as an unsent edit from the start.**
- Location: `board.html`, `fill` and `isDirty`.
- `baseline` is the raw entry value, while a textarea's `value` turns `\r\n` into `\n`, and a text input strips newlines.
- Case: a body with CRLF. `isDirty` is true immediately, so `panelFor` never refills the panel from live updates and its revision goes stale.
- Any save also rewrites the body, or a multi-line title or brief, although the operator did not touch it.

**F3. MEDIUM, design: why the spawned server failed is discarded.**
- Location: `internal/boardcli/open.go` `spawnServer`; `internal/boardcli/cli.go` `serveCmd` and `PersistentPreRunE`.
- `spawnServer` redirects only stderr to the server log; stdout is nil.
- `serve` reports every failure on stdout: config load or resolve in `PersistentPreRunE`, and `outputError(out, …)` for a `Run` error (register, listen, state write).
- Case: a broken `board.yaml` in the prime. The child exits, and the log stays empty or absent of any cause. `open` waits 15 s, then says the log "opens with its pid: end that process if it hangs".

**F4. BLOCKING, design: the watch loop has no one-second floor.**
- Location: `internal/boardserve/run.go`, `watchBoard`.
- Every `events` receive runs a full iteration at once. One board write renders many files in the watched directory, so the loop reads `board.json` and stats the binary many times within milliseconds.
- This breaks `PATTERN-daemon-wakeups` (at most once a second).
- It also defeats `Replaced`'s two-sighting check (`selfexec` `changedTwice`): two sightings can be microseconds apart, in the middle of a deploy.
- The decision record's carry-over lists this as `watch-loop-no-one-second-floor`, and it is still unfixed.

**F5. MEDIUM, design: an unsent edit is thrown away when the entry's kind changes.**
- Location: `board.html`, `panelFor` (`panel.kind === entry.kind`).
- Case: the operator has typed in a note's form, and another writer promotes the note. The next `render` builds a new panel, so the typed text is lost.
- On a stale (409) save, the fresh-entry block and the message are lost with it. D10 requires the unsent edit to stay in its form.
- The operator's own promote click with a dirty form loses the edit the same way.

**F6. LOW, design: the page's promote drops the board's warning.**
- Location: `internal/boardserve/write.go`, `promote`.
- `PromoteTo`'s result is discarded, including `RecordWarning`.
- Case: a Sketch whose body changed after its record was stored. The CLI warns and names `lym board upsert --record-file`. The page reports plain success while the raddle run is released.

**F7. LOW, design: a failure of `serving.Serve` is never logged.**
- Location: `internal/boardserve/run.go`, the `Serve` goroutine.
- An accept error only cancels the watch. The log says `exiting: stopped`, and `Run` returns nil.

**F8. LOW, design: `open` does not recover when the lock holder exits while it waits.**
- Location: `internal/boardcli/open.go`, `runOpen`.
- Case: `live` is true, but the server is just closing its listener (idle or replaced exit) before releasing the lock. Probes fail, the lock then reads free, and `open` never spawns.
- It refuses after 15 s and tells the operator to end a process that has already gone. The window is narrow.

**F9. LOW, design: a `serve` run by hand leaves the log without its pid.**
- Location: `cli.go` `serveCmd` (`Log: os.Stderr`) and `run.go` `Run`.
- `Run` truncates `server.log` and then writes the pid line to the terminal. A later `open` refusal says the log opens with the pid, but the log is empty.
- The plan does say "stderr as the log"; D11 and the refusal assume that stderr is the log file.

**F10. MEDIUM, design: an action button lets a dirty form overwrite another writer unrefused.**
- Location: `board.html`, `send` (the success branch, `panel.revision = answer.entry.revision`).
- Case: the form holds a typed title at revision R1. Another writer changes the title (R2), and the form correctly keeps R1. The operator clicks `draft`, which is sent with `entry.revision`, R2. On success the dirty panel takes R3.
- Save then sends the typed title with R3 and silently overwrites the other writer's title, with no stale refusal and no fresh entry shown.

**Outside my lens, one sentence:** `showFresh` omits project and dependencies, so a conflict in either is invisible beside the unsent edit.

**Tool calls**
1. `markBeamWorkspace`, a search for `fsx/atomic*.go` and for the `depends_on` upsert case: the mark insertion and the write helper were not in the manifest; the file glob matched nothing.
2. `fsx.go` `AtomicWriteBytes`, `depends_on` references and the `Store.UpsertTask` head: needed the state and launcher temp-file mode and the upsert's field handling, neither of them read before.
