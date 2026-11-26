```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each Decision's mechanism step by step against the code and checked the end state against the Decision's own purpose.

**List: each decision, the case used, purpose met or not**
- D1: `lym area start` on Neve, then a pane runs `busctl --user status`. Met, per the probe's run 2.
- D2: the operator adds `~/.config` to `write`, then runs `lym area start`. Met: the start refuses.
- D3: a cold hub. `lym area start` → `supervise` → `boot` → `lym area exec -- tmux`. **Not met**: no step builds the server's environment or its socket argv (F2).
- D4: a pane connects to `/run/user/1000/bus` through a symlink. Met, per the probe.
- D5: a pane connects to the creel socket of hub B, which is area-aware. Met.
- D6: the operator runs host-side `lym reed attach` on an area hub. **Not met**: host-side code runs before the ruleset applies (F1).
- D7: `lym board list` from a host shell in an area hub. Met for root steps. **Not met** for the exempt `reed attach` (F1).
- D8: an area pane posts to `B:orch`. A's outbound `deliver` goes through the supervisor, then B's `confirm` goes back to A's socket, found from the machine record. Met.
- D9: a gate's `RunMemoryCapped` run is OOM-killed inside the area. **Not met**: the scope kill fails (F4).
- D10: a codex seat launches inside the area. **Not met**: the home still links into `~/.codex` (F3).
- D11: the background pusher runs `git push` over HTTPS. Met: `gitexec.childEnv` passes `GIT_CONFIG_*` through (`internal/gitexec/gitexec.go:373-384`).
- D12: a prime bring-up inside the area calls `EnsureRelay`, which goes to `slack-relay start`. Met. The dropped `runproc` record departs from today's registration (F6).
- D13 `window`: Konsole runs host-side `lym reed attach`. **Not met**: same as F1. D13 `deploy`: the operator runs `lym area deploy` on a pushed commit. **Not met against its bound** (F5).
- D14, D15, D16: no runtime mechanism to walk, or D15 is unmeasured by its own statement. Nothing to report.

**Findings**

F1 — BLOCKING, design. D6, D7 and D13 `window`.
Host-side `lym reed attach` is exempt from D7, and it is also the command D13's `window` runs. Before D6's ruleset applies at the client's `exec`, it runs code an agent chooses, on the host:
- `CheckTmuxFloor(c.eng.TmuxPath())` runs `tmux -V` with the hub's reed config key `tmux`.
- `ensureWatchdogSpawned` starts a long-lived host-side `lym reed watchdog --tmux <cfg> --shell <cfg>`.
- `EnsureSession` reads hub state.

An agent writes the hub layer's `reed.tmux` as a script under the hub, and the operator's next attach runs it outside the area. D7's "an exempt verb skips those steps' hub reads" covers only the root steps, not the verb's own reads. D3's "argv from `PATH` or the file's `tmux`, never from hub config" covers only the supervisor.
Evidence: `internal/reedcli/attach.go:57` and `:92`; `internal/reedengine/lock.go:113-114`; `internal/reedengine/config.go:17`; `internal/reedengine/spawnwatchdog.go:58`.
Fix: before any hub read, attach takes the tmux path from `PATH` or the area file, ignores the reed config, and spawns no watchdog outside the area. Add an acceptance criterion for it.

F2 — MEDIUM, design. D3 and D6.
Today reed spawns the server after its env hygiene: `e.scrub.Apply`, `stripTraceID` and `stripIdentityEnv`. In an area hub, the supervisor spawns the server from the operator's host-shell environment, and no Decision gives it that hygiene. The Constraint "reed's server spawn scrubs both" no longer holds on that path.
No step is named to:
- set `TMPDIR`, `GOTMPDIR` and `TMUX_TMPDIR` to the area root, which D6 states as a fact inside the area;
- put the `-S` socket path in the argv the supervisor builds, which reed's `Socket()` and the host attach must agree on.

Every pane then inherits the host environment: `DBUS_SESSION_BUS_ADDRESS`, `SSH_AUTH_SOCK`, and any `GH_TOKEN` or `GITHUB_TOKEN`. `githubclient` tries `GH_TOKEN` before every other source, so the operator's token reaches the area, against D11's purpose.
Evidence: `internal/reedengine/lifecycle.go:461-477`; `internal/githubclient/doc.go:11`.

F3 — MEDIUM, design. D10, codex.
`codexengine` links the lym Codex home's `auth.json` and `sessions` into the operator's own Codex home. The home itself comes from `codex.yaml`'s `home` key, which is hub config. Inside the area `~/.codex` is unreadable (D2, D6), so the links dangle and D10's "logged in once inside the area" cannot hold.
D10 says Claude ignores the env module's key, but says nothing like it for codex: not that the links go, and not that `codex.yaml`'s `home` is ignored. The parity that D10's own Rationale cites fails.
Evidence: `internal/shuttleengine/codexengine/home.go:135`; `internal/shuttleengine/codexengine/doc.go:4-7`.

F4 — MEDIUM, design. D9, kind run.
After a SIGKILL or an OOM kill, `runMemoryCapped` writes `cgroup.kill` under `/sys/fs/cgroup` to end what the command left in its scope. D6 grants `/sys` read only, so the kill fails inside the area and the command's children outlive it.
Today the scope's cgroup also comes from the marker pipe that `scopeStartScript` writes under `systemd-run`. In D9's held-child flow, no step produces that marker or reports the scope's cgroup. D9 covers only "the OOM watch reads its cgroup as today".
Evidence: `internal/proc/memorycap_linux.go:146-200` and `:249-262`.

F5 — MEDIUM, design. D13 `deploy`.
`lym area deploy` runs the area file's deploy command in a clone of a commit that agents pushed with the area's token. Batten's deploy is `tools/deploy` from the repository (`landlock-area-2026-11-23.md:141`), so code the area wrote runs on the host while the deploy runs, not only once the result is installed.
That contradicts D13's bound "no verb runs area-chosen code on the host". D16's non-claim covers only "landed lym code running on the host once deployed".
Fix: state the bound as it is, or narrow the deploy command so it runs no repository code.

F6 — LOW, consistency. D12.
D12 says the relay "holds no `runproc` record". Today the relay registers under `runproc.HubOwner`, and the background of `PATTERN-child-processes` says so: "the Slack relay sits under `runproc.HubOwner`". The record's declared `PATTERN-child-processes` change does not drop that.
Evidence: `internal/runproc/doc.go:9`; `internal/runproc/record.go:48-49`; `internal/runproc/end.go:100`; `pattern/PATTERN-child-processes.md:40`.

F7 — MEDIUM, consistency. D6.
D6 sets `TMPDIR` inside the area to the area root, which is not a `lymtemp` run directory. `PATTERN-lym-files`'s temp-root rule says "No code hands a child a `TMPDIR` that is not a run directory", and the record declares no change to that entry.
The mechanism itself works: lym's root becomes `<area root>/lym-<uid>`, and the creel socket sits directly in it. Under the review's PATTERN rule this may be BLOCKING; that is the handler's call.
Evidence: `pattern/PATTERN-lym-files.md:68-70`; `internal/lymtemp/doc.go:1-13`.

**Tool calls**
1. `reedengine` `TmuxPath`, `CheckTmuxFloor`, the source of `cfg.Tmux`, and `ensureWatchdogSpawned`: the manifest did not show where attach's tmux path comes from or what it spawns.
2. `SpawnWatchdog`, `killScope`, `gitexec`'s env, the relay's `runproc` role and codex's home links: one hop past D6, D9, D11, D12 and D10's cited code.
3. `gitexec.childEnv`, its doc, and `RoleSlackRelay`'s registration: to confirm `GIT_CONFIG_*` reaches git and where the relay is recorded.
