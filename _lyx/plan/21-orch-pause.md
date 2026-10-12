# Card 21 — orch-pause

**Create:**
- `internal/orchengine#PauseWatch`
- `internal/orchengine#Watcher.WatchPause`
- `internal/orchengine#Watcher.pauseTick`
- `internal/orchengine#TestWatcher_HubPauseInterruptsAtGraceEndHoldsAndQueuesTheResumeLine`
- `internal/orchcli#runnerSession.Interrupt`

**Edit:**
- `internal/orchengine#Session`
- `internal/orchengine#State`
- `internal/orchengine#ResetForFreshLaunch`
- `internal/orchengine#Watcher`
- `internal/orchengine#Watcher.tick`
- `internal/orchengine#fakeSession`
- `internal/orchengine#watchEnv`
- `internal/orchengine#watchEnv.newWatcher`
- `internal/orchcli#orchCLI.watchCmd`
- `internal/orchengine/doc.go`
- `internal/orchengine#orchengineAllowedImports`
- `PATTERN.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Cover the orch under a hub entry through the orch watcher alone (D7), keeping it the single writer into the orch pane.

- `Session` gains `Interrupt(guid string) error`; orchcli's `runnerSession.Interrupt` delegates to `Runner.Interrupt`, shuttle's interrupt choreography with its pane check.
- `PauseWatch` is the told `RecordPath` and `Alive func(pid int, startTime string) bool`; `WatchPause` sets it, and a watcher never told one reads no record. `orchCLI.watchCmd` tells it `hubgeom.PauseGeometry` of the prime and `parentlock.ProcessAlive`.
- `tick` calls `pauseTick` after the events read and before the phase switch; `held` true returns from the tick there, so while a live hub entry stands no notice is delivered and no context cycle starts or advances, however long the pause lasts. A slug entry never covers the orch.
- `State` gains `LaunchedAt` (JSON `launched_at`), set by `ResetForFreshLaunch` from its `launchedAt` argument, which every fresh launch and adoption already passes; `Watcher` gains the time of its first tick, in memory.
- `pauseTick` under a live hub entry, once per entry (in memory, keyed by `PausedAt`):
  - an entry whose `Caller` parses (`agentname.Parse`) as the orch role in the prime is the orch's own pause: nothing is interrupted;
  - an entry that already owns the orch's session (`Record.Owner(st.Strand)`) was handled before a restart: nothing more;
  - an entry whose `PausedAt` is not after `st.LaunchedAt` is the orch started under the entry: nothing is interrupted, nothing is recorded and no resume line goes; a zero `LaunchedAt`, from an older state file, reads as launched before every entry;
  - an entry whose `GraceEnd` lies before the watcher's first tick, and that does not own the orch's session, is marked handled in memory with no Esc, so a watcher restarted after the grace end never interrupts and queues no resume line;
    an entry the watcher was already running for, including one whose `GraceEnd` equals `PausedAt`, goes to the bullets below even when the first tick that reads it falls after `GraceEnd`;
  - before `GraceEnd`: when the idle probe passes and `pauserecord.ListNetOps` lists no operation whose `Strand` parses as the orch, the orch is left alone and the entry is done, since there is nothing to interrupt; otherwise it waits;
  - at or after `GraceEnd`, when the idle probe still reads busy (a local command with no network operation included): under the record lock it checks the entry is still live and that no live entry owns the orch, records `Session{Orch: true, GUID: st.Strand}` in it, and then calls `Interrupt`; it reads the probe once more and interrupts once more when still busy, so at most twice. An entry gone at that point sends nothing.
- `pauseTick` with no live hub entry looks for a resumed entry owning the orch's session with `ResumeNoticed` false: it queues `pauserecord.ResumeLine` through `QueueNotice`, then sets `ResumeNoticed` under the record lock, so the line goes once and is delivered idle-gated by `deliverNotice`. An orch the entry does not own, idle at pause time or the caller, gets none.
- An unreadable record holds nothing and logs one Warn per change of error.
- `orchengineAllowedImports` gains `github.com/Knatte18/loomyard/internal/pauserecord` and `github.com/Knatte18/loomyard/internal/agentname`.

`PATTERN.md`'s orch-pane entry becomes:

```
- `PATTERN-orch-pane-single-writer` — Typing into the orch session from Go: only the orch watcher does it, idle-gated except the interrupt a hub pause asks for; another module queues a notice through `orchengine` instead.
```

Bound: the watcher sends Esc only into a busy orch, only at a live hub entry's grace end, at most twice per entry, never when the orch is the entry's caller or was launched at or after the entry's `PausedAt`, and only after recording the orch in the entry.

Test: the new table test drives `watchEnv` over a temp record (`watchEnv` gains its path; `newWatcher` tells it) with `fakeSession` counting `Interrupt` calls and a scripted idle probe: a busy orch is interrupted at the grace end and not before, twice when the probe still reads busy, and is recorded in the entry; an idle orch with no orch marker is never interrupted or recorded; an idle orch with an open orch marker is interrupted only if it reads busy at the grace end; an entry whose caller is the orch interrupts nothing; a watcher created after a live entry's grace end over a busy orch makes no `Interrupt` call, still holds notices while the entry is live, and queues no resume line after the resume; a watcher created before the entry, with the entry written at grace 0 and the first tick reading it after `GraceEnd`, interrupts a busy orch and records it; a state whose `LaunchedAt` lies between the entry's `PausedAt` and `GraceEnd`, with a busy probe, gets no `Interrupt` call at the grace end and no resume line after the resume; a queued notice and a cycle request wait while the entry is live; after the entry is resumed the resume line is queued once and delivered on an idle tick, and an entry that never owned the orch queues none.
`fakeSession` gains `Interrupt`; `doc.go` gains a "Hub pause" section; the budget row grows by one.

**ImpactSummary:** orchengine's Session seam gains one method (its fake and the one production adapter on this card) and the watcher a told pause reading; nothing changes while no hub entry is live.
