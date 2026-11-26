```yaml
lens: interactions
outcome: reported
```

Interactions lens: I checked pairs of decisions that act on the same state.

**Pairs checked**
- D2 × D6 (write-list check against the derived grants): broken (F5)
- D2 × D6 (`lym area init` content against D6's `/tmp` rule): broken (F7)
- D3 × D4 (supervisor death, then `lym area stop` and `lym area start`): broken (F4)
- D3 × D7 (`lym area start` and reed spawn from outside against the host-side refusal): held
- D3 `lym area stop` × D7 rationale (host lym reading agent-written state): broken (F6)
- D3 inside test × D6 (supervisor record read grant) × D8 (cross-hub inside test): broken (F8)
- D3 × D12 (subreaper, relay as the supervisor's child, inside test): held
- D4 × D6 × Constraints-Yama (supervisor deciding the attach client's connects): broken (F1)
- D5 × D6 × D8 (socket path and level derived from the machine record): broken (F3, F9)
- D5 × D15 (allow list read at each decision against a profile frozen at root load): broken (F10)
- D6 area root × D3 and D4 (supervisor socket, attach listener handover): broken (F2)
- D6 and D7 attach exemption × D3 and D7 rule against host-side reads of hub config: broken (F11)
- D6 Go caches × D13 deploy with the host's toolchain: broken (F12)
- D7 cwd-scoped check × D13 hibernate and machine-wide host verbs: broken (F13)
- D8 × D3 (in-area daemon skips its record; outbound worker spawn rules): held
- D8 × D12 (relay outside posting to the in-area daemon): held
- D9 × D6 (`RunMemoryCapped` scope kill against the `/sys` read-only grant): broken (F14)
- D10 × D2 (config directories moved under the hub against the write-list check): held
- D10 × the Constraints line on reed's scrub of `CLAUDE_CONFIG_DIR`: held
- D11 × D2 (token file read-only, write-list check): held
- D11 × D13 deploy (pushed commit against the operator's command): held, through D16's non-claim
- D13 window × D6 attach (supervisor-started Konsole is the supervisor's descendant): held for the window, broken for a host-shell attach (F1)
- D14 × D8 (creel identity before the boot): held
- D15 × D3 inside test (profile label): held
- Acceptance criteria D6 attach, D5 and D2 against the decisions above: broken where F1, F3 and F5 apply

**Findings**

F1 — BLOCKING, design, decision-record.md D6 (attach client) × Constraints (Yama) × D4.
The operator's host-shell `lym reed attach` is the shell's child, not a descendant of the supervisor.
Under `ptrace_scope` 1 the supervisor cannot read its memory or take its socket.
So every connect the attach client makes, its own tmux socket included, is refused, the same way the measured orphan was.
The attach acceptance criterion cannot pass.
Evidence:
- decision-record.md:123 and :250
- sandbox-without-new-kernel-2026-11-23.md:82, :90-93
- seccomp-connect-probe/main.go:316-317 and :350-356 (`/proc/<tid>/mem`, `pidfd_getfd`)

F2 — BLOCKING, design, D6 area root × D3 and D4 listener handover.
The supervisor socket lives under the area root, where agents hold every right (D6:115-119).
An agent can unlink it and bind its own listener there.
A host-side attach (D6:123) then hands the operator's client filter listener to the agent.
The agent answers CONTINUE, and the client's `detach-client -E` command connects freely, to the bus included, and escapes.
Host `lym area start` likewise asks a fake `boot` (D3:63).
No decision says who checks that the socket is the supervisor's.
Evidence: decision-record.md:63, :91, :115-119, :123-124.

F3 — MEDIUM, design, D5 × D6 × D8 (machine record).
D5 admits another hub's creel socket by the path its machine-record entry carries (D6:120-121), read at each decision.
Any process outside an area, such as any agent of a hub without one, can write that shared file.
An entry naming `/run/user/<uid>/bus` with the area-aware level reaches the allow list, which contradicts D5's "never a bus".
No decision checks the path's shape.
Evidence: decision-record.md:100-101, :120-121, :147; hubregistry.go:37-44, :55-59.

F4 — MEDIUM, design, D3 stop × D4 supervisor death.
After the supervisor dies, its tree is reparented to `systemd --user`.
`lym area stop` "ends every process left in the supervisor's tree", but that tree no longer exists.
dobby and the creel daemon live on with a dead listener: every connect fails with `ENOSYS` while they hold their locks.
The next `lym area start` boots beside them.
D4's way forward, stop then start, is not a defined end state.
Evidence: decision-record.md:72-73, :94-95.

F5 — MEDIUM, design, D2 × D6.
D2's bound says no host-side input is writable inside.
Its start check covers only the area file's `write` list, while D6 grants every right on the derived hub root and the Go caches.
A hub root at or above `$HOME`, or one holding a host input, passes unchecked.
The D2/D5 criterion tests only the file.
Evidence: decision-record.md:52-54, :115, :287.

F6 — MEDIUM, design, D3 `lym area stop` × D7 rationale.
Host-side `lym area stop` ends processes through `runproc` records that live in the hub's `.lym`, which agents write.
An agent can list any host pid there and have the host-side stop signal it.
D7 forbids host lym acting on agent-written hub state, and `lym area` is exempt from D7.
Evidence: decision-record.md:73, :131-137; PATTERN-child-processes.md:39-41.

F7 — MEDIUM, consistency, D2 × D6.
D2 has `lym area init` write the Landlock study's read list.
That list grants every right on `/tmp`, `~/.cache/go-build` and `~/go/pkg/mod`, and read on `~/.claude` and `~/.claude.json`.
This contradicts D6: `/tmp` gets no write, and the area uses its own config directories.
D2's start check refuses none of these entries.
Evidence: decision-record.md:51, :117; landlock-area-2026-11-23.md:89-92.

F8 — MEDIUM, design, D8 × D3 × D6.
For a peer from area C, D8's daemon applies D3's inside test, which needs C's supervisor record.
An area hub's own in-area daemon is granted read on its own supervisor record only (D6:114), so it cannot recognise a peer from another area.
No decision says how a daemon finds every area hub's supervisor.
Evidence: decision-record.md:59-60, :70-71, :114, :145.

F9 — MEDIUM, design, D8 × D6 machine record.
`hubregistry.Record` rewrites the whole file through a typed `Entry` that has no socket-path or level field.
Any daemon of an older binary that records itself therefore strips other hubs' socket paths and area-aware levels.
Area hubs drop off the allow list until the next `lym area start`.
D8 handles older binaries only as receivers.
Evidence: hubregistry.go:37-44, :55-59; decision-record.md:147-149.

F10 — MEDIUM, design, D5 × D15.
D5 re-reads other hubs' creel sockets from the machine record at each decision.
D15's profile is rendered from "the same allow list", named by a hash of its text, and loaded once by root.
Either the hash changes whenever a hub records itself, and the start refuses until IT reloads the profile, or cross-hub creel is frozen at load time.
Neither case is stated.
Evidence: decision-record.md:100-101, :223-227.

F11 — BLOCKING, design, D6 and D7 attach exemption × D3 and D7 rule.
D7 exempts `lym reed attach`, and D6 restricts only the client it `exec`s.
Before that, attach runs on the host:
- `CheckTmuxFloor` spawns `tmux -V` from reed's hub-config `tmux` key;
- `EnsureSession` and `Status` read `.lym/reed.json`;
- `SpawnWatchdog` starts a detached host-side watchdog with the hub's `tmux` and `shell`.

reed's config classifies both keys `CategoryExecutable`, and agents write every hub layer.
So a host attach, including the one D13's `window` runs, executes agent-chosen binaries outside the area.
This contradicts D3:66 ("never from hub config") and D7's rationale.
Evidence:
- reedcli/attach.go:57, :70, :87
- reedcli/spawnwatchdog.go:12
- reedengine/config.go:64-65
- decision-record.md:66, :123, :132, :199

F12 — BLOCKING, design, D6 × D13 deploy.
D6 gives agents every right on the Go build and module caches.
D13's `lym area deploy` builds on the host "with the host's toolchain", which uses the same caches.
An agent can plant a build-cache or module-cache entry that the host build links into the installed `lym`.
That is code persisted from the area that later runs outside it, against the Goal.
D2's protected inputs name the Go toolchain but not its caches.
Evidence: decision-record.md:53, :115, :203-204, :6.

F13 — MEDIUM, design, D7 × D13 and host-side machine verbs.
D7 refuses only a verb whose cwd resolves to the area hub.
`lym hibernate request` "pauses every session as today", and `lym machine start` brings back every hub the machine record lists.
Run from a host shell elsewhere, both act over area hubs, reading their agent-written state and starting their processes outside the area.
D7 names no hub-by-record route.
Evidence: decision-record.md:131-134, :201; machinecli/cli.go:151; machinecli/hubs.go:74.

F14 — MEDIUM, design, D9 × D6.
At a kill or OOM at the cap, `RunMemoryCapped` ends the scope's leftover tree through `killScope`, which writes the scope cgroup's `cgroup.kill` under `/sys`.
D6 grants `/sys` read only, so inside an area the capped gate's leftover processes survive.
D9 routes only the scope's creation through the supervisor.
Evidence: memorycap_linux.go:193-204, :249; decision-record.md:114, :156-163.

Tool calls:
1. `ensureWatchdogSpawned` and reed's `tmux`/`shell` key categories, to confirm the host-side spawns attach makes and that the keys are executable; the manifest did not cover them.
