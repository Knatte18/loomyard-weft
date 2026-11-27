```yaml
lens: code-affected-set
outcome: reported
```

Restating: code-affected-set lens, covering what the diff forces outside the lines it touches.

**List: per change, the set found, covered or not**

| Change | Set found | Covered |
|---|---|---|
| `isRunAgentSession` → `IsRunAgentSession` | `checkTaskCreation`, `UpsertTaskWithRecord`, no other references | yes |
| `writeAttachTask` → `writeWorktreeTasks`, `assertAttachTaskOnly` → `assertWorktreeTasks` | no stale references outside a research data dump | yes |
| `BuildWorkspace` gains `members` | `regenerateHubWorkspace` (both calls), `TestBuildWorkspace`, `fabricengine` doc | yes |
| `InstallWorkspaceContent` hook | `cmd/lym/main.go`, `prerunseams_test`, ide hub checks, `docs/overview.md` | yes |
| Render adds `board-data.js` and `board.html` | manifest, owner-set write, `FabricReplay.Claims` and its comment, `boardengine` doc, `render.go` header | yes |
| same | `docs/overview.md` **board** module section | no (F4) |
| New embedded asset `board.html` | `.gitattributes` LF pinning of `go:embed` targets | no (F3) |
| `Outputs.Labels` | `Config.Outputs`, `TestOutputs` | yes |
| `rerender` `Short` | no doc or test quotes the old text | yes |
| New verbs `open` and `serve` | `boardcli` doc, `Command`, audience annotations | yes |
| same | `docs/overview.md` **board** module section | no (F4) |
| New opener spawn | PATTERN-child-processes: spawn log yes, teardown log no | no (F2) |
| New unbounded watch loop | `daemonwakeups_test` list and background list yes, the entry's one-second floor no | no (F1) |
| `RoleBoardServer` | `runproc` doc, `subjectPassSpared`, `end_test`; the anchor move's `EndSparing` and `liveUnsparedChild` need no change | yes |
| `BoardServeDirName` | `Paths`; no structural list needs it | yes |
| `AcquireWriteLockWithin` tries once first | doc comment, test row | yes |
| VS Code tasks and keys | file headers, `docs/overview.md` VS Code passage | yes |
| PATTERN changes | none declared; the daemon-wakeups list extension landed | yes |
| Commit trailers | the `Pattern-Departure` and `Loosened-Check` allowlist entries are all removed again within the range | yes |

**Findings**

**F1. BLOCKING, design.** `internal/boardserve/run.go` (`watchBoard`)
- The `case _, open := <-events:` arm has no one-second floor, so every `fswatch` event wakes the loop at once. This breaks PATTERN-daemon-wakeups: "wakes at most once a second".
- The watch covers the whole board directory (`opts.Watch(opts.BoardPath)`, no names), so one render's many file writes cause a burst of wakes, each re-reading and hashing `board.json`.
- The same burst runs `opts.Replaced()` back to back. `selfexec` reports a replacement after two consecutive matching reads, so a binary still being copied can pass that test within microseconds.
- The record's carry-over block lists `watch-loop-no-one-second-floor`, and it is still unfixed.
- Evidence: `internal/boardserve/run.go` `watchBoard` loop; `pattern/PATTERN-daemon-wakeups.md` "One-second floor"; `internal/selfexec/selfexec.go` `changedTwice`.

**F2. BLOCKING, design.** `internal/boardcli/opener_linux.go` and `opener_windows.go` (`startOpener`)
- The opener is waited on through `cmd.Wait`, but only its spawn is logged; no exit or teardown is.
- PATTERN-child-processes says a path that starts a process "logs its teardown wherever it waits for one". Only a `proc.Reap` spawn is excused from that.
- The recorded departure from `proc.Reap` does not cover the missing log.
- Evidence: the `go func() { exited <- cmd.Wait() }()` line in both files; `pattern/PATTERN-child-processes.md` "Spawn observability".

**F3. MEDIUM, design.** `.gitattributes`
- `internal/boardengine/board.html` is a new `go:embed` target with no `text eol=lf` line (`git check-attr` answers `eol: unspecified`). The file's own rule pins embed targets so the built bytes do not depend on `core.autocrlf`.
- A CRLF checkout would build a binary whose `board.html` differs from an LF build's. Machines sharing a hub would then rewrite the committed file on each other's writes.
- The record's carry-over block lists `board-html-not-pinned-to-lf`, and it is still unfixed.
- Evidence: the header comment of `.gitattributes`; `internal/boardengine/viewer.go` `//go:embed board.html`.

**F4. LOW, consistency.** `docs/overview.md`, the **board** module section (about lines 462-495)
- The section describes the board's verbs and rendered output, but says nothing of `lym board open`, `lym board serve`, the viewer page, `board-data.js` or `board.html`.
- The diff adds only a package-tree line and the VS Code task and key text.
- No card asks for this text, and I found no repository rule that requires it.
- Evidence: `docs/overview.md` lines 464-495 as the branch leaves them.

**Tool calls (5 of 6)**
1. Grep for the old symbol names and for docs naming render outputs, tasks and keys: the manifest had no repo-wide references.
2. Grep for consumers of runproc roles, hub `.lym` dir names, board verb lists and board file lists: these callers were not in context.
3. Grep for `tasks.json` and keybinding docs, `eol=lf` tests and runproc record readers: these were not in context.
4. Spans of `internal/fabriccli/anchordeps.go` and a grep of role use: this reaches a process-record consumer one hop beyond `RoleBoardServer`.
5. `docs/overview.md` board section, the `fabriccli` doc's anchor refusals and `EndDobbies`: none of these text spans were in context.
