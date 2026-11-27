```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff `24688edd..HEAD`. I traced these cases by reading the code and ran nothing.

**Cases run**

| Function or branch | Case | Result |
|---|---|---|
| `migrateLoaded` / `boardCriticalSection` | `Expecting` absent or set; refusal before the mutation | correct |
| `Board.Expecting` / `checkExpected` | create of a new, existing or project slug; removed; changed; unmigrated entry | correct |
| `EntryRevision`, `BoardRevision`, `Board.View` | absent `board.json`; unknown keys; load without save | correct |
| `buildView`, `sectionName(s)`, `designFileName` | empty board; no project; done project; empty body; missing decision record | correct |
| `renderFiles`, `boardDataScript`, `unmetDependencies`, `afterCell`, `Claims` | missing dependency; handle-less dependency; `</script>` in a body | correct |
| `ViewerPage`, `Config.Outputs`, `IsRunAgentSession` | empty inputs | correct |
| `ServeHTTP` checks, CSP, `serveFile` | wrong Host or token; empty name; name with a separator | correct |
| `serveBoard` / `awaitChange` / `BoardChanged` / `beginDrain` | empty revision; cancelled request; drain called twice | correct |
| `serveWrite` / `decodeWriteRequest` | `null` body; unknown key; oversized body | correct |
| `applyWrite`, `upsert`, `create`, `promote`, `changeStatus`, `statusAfter` | empty revision; nil fields; removed entry; project; run status | refusals correct; warnings dropped (F3) |
| `Run` | held lock; `Register` or listen failure; exit drain | lock released correctly; the error is lost (F1) |
| `watchBoard` | watch failure; closed channel; event burst; idle; replaced | correct |
| `Live`, `ReadState`, `writeState`, `WriteLauncher`, `Probe`, `hardenFile` | missing directory; leftover state | correct |
| `runOpen` / `awaitServer` / `openLauncher` | free lock; held lock; cap zero or non-zero; opener fails | correct, except a failing server (F1) and an exit between reads (F5) |
| `spawnServer`, `startOpener`, `watchDirectory`, the `open` and `serve` RunE | start failure; `--board-path`; run agent | server stdout discarded (F1) |
| `workspaceMembers`, `regenerateHubWorkspace`, `BuildWorkspace` | no hook; refused member; unreadable beam anchor; nil members | correct |
| `WorkspaceContent`, `WorkspaceTasks`, `writeWorktreeTasks`, key block | `LymBinary` error; empty beam | correct |
| `board.html` `write` | action succeeds while an edit form is dirty | wrong (F2) |
| `board.html` `send` / `retrying` | response lost after the write; double click | misreported (F4) |
| `board.html` `markdown`, `inline`, `safeTarget` | `javascript:`; `//host`; code span as a link target | safe; code-span target is wrong (F6) |
| `board.html` `poll`, `draw`, `buildForm`, `drawRemoved`, filters, `age` | empty board; removed entry; unchanged answer | correct |

**Findings**

**F1. MEDIUM, design. A server's startup error is discarded.**
- Location: `internal/boardcli/open.go` (`spawnServer`) and `internal/boardcli/cli.go` (`serveCmd`, PersistentPreRunE).
- `spawnServer` sets only `cmd.Stderr = log`, so stdout goes to the null device.
- Every `serve` failure goes to stdout through `outputError` or `output.Err`. This covers a `board.yaml` that fails to load, and a failed `Register`, `net.Listen`, `NewServer` or `writeState`.
- Concrete case: `Register` fails. `open` waits 15 s and says "did not answer; its log opens with its pid: end that process if it hangs". The log holds only the pid line of an exited process, with no cause.
- A failure before `Run` takes the lock (config or cwd) is worse. The log is not truncated, so it still opens with a former server's pid, and the operator is told to end that pid.
- Evidence: `open.go` `spawnServer` (`cmd.Stderr = log`, no `Stdout`); `cli.go` `serveCmd` (`return outputError(out, err.Error())`); `run.go` `Run` (truncation comes after the lock).

**F2. MEDIUM, design. A successful action silently moves a dirty edit form onto the new revision, so a later Save can overwrite another writer.**
- Location: `internal/boardengine/board.html`, `write()`, around lines 353–356.
- Sequence:
  1. The operator types a new title in the edit form.
  2. Another writer changes that title. The poll updates the view, and the dirty form correctly keeps the old revision.
  3. The operator presses `release` or `draft`, which sends the current `entry.revision` and succeeds.
  4. The success branch sets `state.revision = answer.entry.revision` and `state.fresh = null`.
  5. Save now passes the revision check and overwrites the other writer's title. No conflict or fresh entry is shown.
- This defeats the stale-write refusal the goal and D8 ask for.

**F3. MEDIUM, design. The server drops the board's record warnings.**
- Location: `internal/boardserve/write.go`, `upsert` and `promote`.
- Both discard the result (`_, err :=`), including `RecordWarning`.
- Concrete case: the page promotes a Sketch whose body changed after its record was stored. The CLI prints a warning naming `upsert --record-file`. The page answers plain success, and the raddle run is released against the outdated record. A body edit of a discussed entry loses its warning the same way.
- D10 says promote acts as the CLI's does.
- Evidence: `write.go` `promote` (`_, err = ...PromoteTo(...)`); `board.go` `PromoteTo` (`RecordWarning`); `cli.go` `promoteCmd` (`envelope["warning"]`).

**F4. LOW, design. Writes are retried or sent twice, so the page reports its own successful write as a refusal.**
- Location: `internal/boardengine/board.html`, `send()` and the action buttons in `entryDetail`.
- `send` wraps a non-idempotent POST in `retrying`. If the write lands but the response is lost, the resend answers "changed since the page read it", or "already exists" for a create.
- Action buttons have no pending guard. A double click makes the first request succeed and the second answer 409, whose message then stays beside the entry.

**F5. LOW, design. `open` never starts a server if the lock holder goes away after the first `Live` read.**
- Location: `internal/boardcli/open.go`, `runOpen` and `awaitServer`.
- Case: the first `Live` is true, then the holder exits (replaced or idle) or fails its start before answering.
- `awaitServer` sees `Live` false on every round, never spawns, and refuses after 15 s with "end that process if it hangs", although nothing is running.

**F6. NIT, design. A link whose target is a code span renders a bogus link.**
- Location: `internal/boardengine/board.html`, `inline()` and `safeTarget`.
- ``[x](`a`)``: the target is the placeholder `\u00000\u0000`. `safeTarget` trims both NULs as C0 controls and returns `"0"`, so the NUL guard misses it and the page renders `href="0"`.

**Tool calls**
1. `grep` of `WorkspaceSettings`, `BuildWorkspace` and neighbouring code in `internal/vscode`. My context lacked whether the settings text is re-encoded before the member splice, which decides whether a trailing `//` comment could swallow the comma. It is re-encoded, so there is no finding.
