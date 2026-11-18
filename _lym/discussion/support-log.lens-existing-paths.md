```yaml
lens: existing-paths
outcome: reported
```

## existing-paths lens report

I traced the existing code that reads or writes the state each decision introduces or changes, and checked whether the decision's result survives it.

### Paths traced

**D1 (machine-layer keys)**
- `configengine` stacks the machine layer below local and above tracked files, with `MachineKeys` derived from `ConfigSpec`: survives.
- The claude module is a degrading `LoadOrTemplate` caller, but degrading applies only to an absent base, so D1's refusal of an invalid value still fails the launch: survives.
- `hubreconcile` never touches the machine file: survives.

**D2 / D3 (account file and pick)**
- The pick points are battenshed's Worktree-Create, beamcli create and the adopt pair-add: survives.
- A task worktree recreated at its old name: the pick's prune runs only inside a pick, and by then the directory exists again, so the old entry stays ("a worktree that already has an entry keeps it"). The old account is still valid, so this is harmless (LOW, below).

**D4 (launch export and scrub)**
- reed `Resume` / `--if-absent` relaunch (`lifecycle.go`, `strand.go`) replays the stored `ResumeCmd` with the frozen export, and `PaneEnv` is re-read each time. The export comes after `PaneEnv`, so it wins: survives, and the record says so.
- `ScrubbedEnv` → `shuttleengines.EnvScrub` → reed server spawn: survives.
- The trust prompt for a new account dir is handled by the existing startup-gate handling in `startup.go`: survives.

**D5 (status line)**
- `--settings` outranks a heddle `.claude/settings.local.json` statusLine: survives.

**D6 (resume copy)**
- Orch resume goes through `Runner.Start` with `ResumeSessionID` (`orchcli/start.go`), so it reaches `Prepare` and gets the copy: survives.
- The sampler and PR-Review resumes also go through `Prepare`: survives.
- Switching back to an earlier account breaks (MEDIUM, below).

**D7 (switch)**
- Shuttle's restart goes through stop and `Start`, which stores a new `ResumeCmd`: survives.
- Other strands of the worktree keep the old account across reed replays: survives as stated.
- The live readers break for those strands (BLOCKING, below).

**D8 (readers)**
- `LocateSessions` (tokencount, sessionusage): survives.
- `AuditForks`, `OutstandingForks`, `SessionNameDrift`, `ProcessLiveness`: break (BLOCKING and MEDIUM, below).

### Findings

**BLOCKING — D8 / D7: the live-session readers read the worktree's current account, not the one the session runs under.**
- `AuditForksIncremental` (`audit.go`), `OutstandingForks` (`background.go`) and `SessionNameDrift` (`sessionname.go`) each derive the transcript path from `claudeProjectDirFor(workdir)` and are passed only `(sessionID, workdir)`.
- D8 has "the live-session transcript read" take the worktree's resolved directory.
- D7 says running strands keep their old account after `lym claude account set`, and after a switch made by a sibling session.
- In both cases the resolved directory differs from the directory the live session writes to. The readers then hit a missing transcript:
  - `OutstandingForks` errors.
  - The fork audit fails with "audit incomplete".
  - Name repair goes quiet.
- That breaks waits on running sessions.
- Fix: read the directory the run was launched under, kept in the run record, or search the known directories by session id.

**MEDIUM — D8: the reader and namer engines are built with no hub `Dirs`, so they cannot load `claude.accounts`.**
- `shuttleengines.SessionNamers()` builds every engine with `configengine.Dirs{}`.
- `RecordReaders()` takes no arguments.
- Without `Dirs` they cannot load the accounts list or the `env` module, so D8's "known directories" for the id-only liveness read, record readers and reed's namer collapse to `~/.claude`.
- The cancelled record had a paragraph on telling these engines the hub's `Dirs`; this record drops it without replacing it.

**MEDIUM — D6: a resume can pick up a stale copy.**
- D6 copies only when the transcript is absent under the resolved directory.
- Example: session X runs on A, switches to B (copied A→B, then grows on B), then the worktree is set back to A. The resume finds A's older prefix copy and resumes it, losing every turn taken on B.
- The resume check (`resume.go`'s `claudeProjectDirFor`) stats only the resolved directory.
- Fix: copy when another known directory holds a newer copy. D8's "most recently written" rule already assumes the newest copy is the true one.

**LOW — D5: `blocked_until` has no writer for opened sessions.**
- Limit hits are classified only in shuttle `Wait` (`hold.go` `usageLimited`, `wait.go`) and in the orch adapter.
- Sessions opened with `lym shuttle open` and standalone strands never record one, so headroom relies on the status line alone there.

**LOW — D2: a worktree recreated under a reused name keeps its old entry.**
- It is not re-picked, so the share skews slightly.

### Tool calls
1. Grep for `.claude.json`/trust writes, orch launch path and usage-limit handling: needed to check D4 trust and the D6 orch resume.
2. Grep for transcript/registry readers in claudeengine, trust and startup files, and the plugins `claudeDir`: needed every current `workdir` reader beyond the agent summaries.
3. Read the `audit.go`, `sessionname.go` and `background.go` reader spans: needed to confirm the readers take `(sessionID, workdir)` only.
4. Read `shuttleengines.SessionNamers` and `RecordReaders`: needed how the reader engines are built and whether they get `Dirs`.
