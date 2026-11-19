# Card 8 — sandbox-reaper

**Create:**
- `plan:internal/reedengine#StopServer` -> `func StopServer(tmuxPath, hubPath string) (bool, error)`
- `plan:internal/reedengine#TestStopServer` -> `func TestStopServer(t *testing.T)`
- `plan:internal/hardenshed#ReapPoint` -> `type ReapPoint string`
- `plan:internal/hardenshed#ReapStart` -> `const ReapStart ReapPoint = "start"`
- `plan:internal/hardenshed#ReapEnd` -> `const ReapEnd ReapPoint = "end"`
- `plan:internal/hardenshed#ReapHalt` -> `const ReapHalt ReapPoint = "halt"`
- `plan:internal/hardenshed#ReapClear` -> `const ReapClear ReapPoint = "clear"`
- `plan:internal/hardenshed#StrayEntry` -> `type StrayEntry struct`
- `plan:internal/hardenshed#ReapDeps` -> `type ReapDeps struct`
- `plan:internal/hardenshed#Reap` -> `func Reap(deps ReapDeps) ([]StrayEntry, error)`
- `plan:internal/hardenshed#TestReap` -> `func TestReap(t *testing.T)`

**Edit:**
- `PATTERN.md`
- `pattern/PATTERN-child-processes.md`

**Extends:**
- `internal/reedengine/doc.go`
- `internal/hardenshed/doc.go`
- `internal/runproc/doc.go`

**Uses:**
- `internal/reedengine#ServerName`
- `internal/reedengine#endSessionByNameVia`
- `internal/runproc#End`
- `internal/runproc#EndProcess`
- `internal/runproc#Owners`

**Intent:**
The sandbox reaper that `lym harden sandbox clear`, every round row and the halts run (D5), and the PATTERN change it needs.

`reedengine.StopServer` kills the tmux server whose socket key `ServerName(hubPath)` derives, by `kill-server` on that key, and removes its socket file, whether or not the hub's directory still exists; it reports false for no live server.
`TestStopServer` is tier2: a server started on a hub path's key is stopped and reported; a second call reports false.

`ReapDeps` carries the root, the row and `ReapPoint`, `ListProcesses` (wired to `proc.ListProcesses`, nil off Linux), `Owners` and `End` (wired to `runproc.Owners` and `runproc.End` over each hub's `fabricengine.RunProcDir`), `EndProcess` (wired to `runproc.EndProcess`), `StopServer`, and the caller's pid.
`Reap`, over the root's hub list:
1. Lists the reed servers of every recorded hub and, on Linux, the processes whose cwd, executable or argv names the root, and returns one `StrayEntry` per server and process found, each naming the row, the reap point, and whether a record names it; a `ReapStart` entry is marked as left by an interrupted row.
2. Ends each hub's recorded processes through `runproc.End` over every owner that hub's records name, sparing the caller.
3. Stops each recorded hub's reed server, its directory present or gone, and prunes the hub's list entry once `StopServer` answers that it stopped the server or that none was live; a `StopServer` error keeps the entry and is returned, joined with the other steps' errors, naming "way forward: run lym harden sandbox clear in the task worktree again".
4. Ends through `EndProcess` only the root-naming processes no record names, skipping the caller's pid and its ancestors, which it reads from the same process listing's parent pids.
5. Removes every entry under the root but the hub list file, listed hubs and unlisted containers alike, so no `.git` under the root is left for fabric's leftover cleaner to refuse.
It touches only servers whose key derives from a recorded hub path and processes naming the root; a hub made outside the root is outside it.
Off Linux the process scan is skipped and only the servers are checked.

The rule change, declared in the overview's `## PATTERN changes`: `PATTERN.md`'s entry line becomes

```
- `PATTERN-child-processes` — Starting or ending a process: log the spawn via `internal/logger`; its starter waits on it, bar a double fork's intermediate; a run's processes register in `internal/runproc` and end only via it, orphans via `EndProcess`; a pinned or spawned lym is named via `selfexec.LymBinary`. — [background](pattern/PATTERN-child-processes.md)
```

and the background file's Run process owner section says `EndProcess` serves the orphans `lym tender reap` ends and the Hardener sandbox reaper's, bounded to processes under a `hardener` run's sandbox root that no record names.
`internal/runproc/doc.go`'s `EndProcess` sentence names the same two users: the orphans `lym tender reap` ends and those the Hardener sandbox reaper ends under a sandbox root.

`TestReap` is tier0 with fakes: a recorded process is ended through `End` and never through `EndProcess`; an unrecorded process naming the root is ended through `EndProcess`; an unrecorded process naming the root that is the caller's ancestor is left; a process not naming the root is left; the server of a recorded hub whose directory is gone is stopped and the entry pruned; a recorded hub with no live server is pruned; a `StopServer` error keeps the entry and returns its way forward; a `ReapStart` entry is marked interrupted; with no `ListProcesses` only servers are reaped; the hub directories and an unlisted container holding a bare repository are gone afterwards.

**ImpactSummary:** One reedengine function, new reaper code in hardenshed and one PATTERN entry's wording; existing reed and runproc callers are unchanged.
