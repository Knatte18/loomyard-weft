```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the existing code paths that meet each decision's new state.

**Paths traced, per decision**

- D3/D6/D7: host-side `lym reed attach`, which D7 exempts: `CheckTmuxFloor(TmuxPath())`, `EnsureSession`, `ensureWatchdogSpawned` and `handOver` all run before any ruleset applies. **Breaks.**
- D13: the `window` verb's Konsole runs `attachLine` → `lym reed attach`, which goes through the same path. **Breaks**, as above.
- D9: `runMemoryCapped` → `killScope` writes `cgroup.kill` under `/sys/fs/cgroup` after a SIGKILL or OOM. **Breaks**, because D6 grants `/sys` read only.
- D9: `scopeHadOOMKill` and `scopeCgroup` only read. **Survives.**
- D10: codexengine `prepareHome` links the lym Codex home's `auth.json` and `sessions` into the operator's Codex home. **Breaks** under an area `CODEX_HOME`.
- D12: the relay registers itself through `runproc.Register(... SelfRecord(RoleSlackRelay))` under `HubOwner`, and `StopRelay` → `runproc.EndRole`. **Breaks** D12's claim that the relay "holds no `runproc` record".
- D5/D6/D8: `hubregistry.Record` rewrites the machine record through `state.UpdateJSON`, which keeps unknown keys on entries it did not update. A new socket-path or level field on another hub's entry survives an older daemon's write. **Survives.**
- D8: when the in-area daemon skips its own `Record`, `RecordedAt` goes stale. `hubregistry.Live` and `entryGone` do not read `RecordedAt`. **Survives.**
- D11: no gitexec code sets `GIT_CONFIG_COUNT`, so the launch-environment override is not overwritten. **Survives.**
- D6: the reed socket path resolves through `$TMUX_TMPDIR` (`socketfile.go:21`), so a pointer at the area root moves the socket as D6 says. **Survives.**

**Findings**

1. **BLOCKING, design, D6/D7 (`decision-record.md` D6, D7).** The host-side `lym reed attach`, which D7 exempts and D13's `window` runs, executes hub-config-chosen code on the host before any ruleset applies.
   - It runs `tmux -V` on the hub `reed` config's `tmux` key through `CheckTmuxFloor`, and it boots through `EnsureSession`.
   - It spawns a long-lived host-side `lym reed watchdog --tmux <hub tmux> --shell <hub shell>`, which never enters the area.
   - The record says the supervisor never takes `tmux` from hub config, but it is silent about this path.
   - Evidence: `internal/reedcli/attach.go:55`, `:67`, `:88`, `:108`; `internal/reedengine/spawnwatchdog.go:58`; `internal/reedengine/config.go:17`.

2. **MEDIUM, design, D9 (`decision-record.md` D9).** Inside the area, the run-cap kill writes `cgroup.kill` under `/sys/fs/cgroup`, which D6's read-only `/sys` grant refuses.
   - After a kill at the cap, what the command left in its scope is not killed.
   - `RunMemoryCapped` returns an error instead.
   - The record says only that the OOM watch "reads its cgroup as today".
   - Evidence: `internal/proc/memorycap_linux.go:198-200`, `:249-256`.

3. **MEDIUM, design, D10 (`decision-record.md` D10).** codexengine's `prepareHome` links the lym home's `auth.json` and `sessions` to the operator's Codex home on every launch, and the record is silent about it.
   - Under D10's area `CODEX_HOME`, those links point outside every grant, so the in-area login and rollouts fail.
   - If the links were granted, they would write the operator's home: the persistence exit D10 closes.
   - Evidence: `internal/shuttleengine/codexengine/home.go:139-145`; `internal/shuttleengine/codexengine/doc.go:6`.

4. **MEDIUM, consistency, D12 (`decision-record.md` D12).** D12 claims the supervisor-started relay "holds no `runproc` record an area process would try to signal".
   - The relay registers itself under `HubOwner` with role `slack-relay` on start.
   - The record sits in the hub's agent-writable run-process directory.
   - `StopRelay` and any `HubOwner` end, including `lym area stop`'s, still go through it.
   - Evidence: `internal/slackcli/wiring.go:394`; `internal/slackengine/start.go:146-147`.

**Tool calls**

1. Grep of the state unknown-key handling, reed's `tmux` key, the watchdog spawn, gitexec's `GIT_CONFIG_COUNT`, the gh cache path, the relay's record, the reed socket path and `killScope`. My context lacked the code one hop past the cited symbols.
2. Read of codexengine `prepareHome`, the gh cache directory, `SpawnWatchdog` and the relay wiring. These are the spawned or linked paths the first grep pointed to.
3. Grep of `hubregistry`'s liveness, to settle whether a stale `RecordedAt` from D8's skipped record matters. My context lacked the code that reads it.
