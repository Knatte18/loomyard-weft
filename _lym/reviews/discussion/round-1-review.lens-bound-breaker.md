```yaml
lens: bound-breaker
outcome: reported
```

**Bound-breaker lens: claims attacked**

| Claim attacked | Result |
|---|---|
| D4 "The supervisor decides every connect" | broken: a nested filter with a listener that answers CONTINUE skips it (F1) |
| D6 attach client: a filter whose listener goes to the supervisor | broken: Yama stops the supervisor reading the client (F2) |
| D6/D3: the supervisor and tmux sockets under the area root are trusted from the host | broken: an agent can replace them (F3) |
| D7 bound: no host-side lym reads agent-written hub config, with `lym reed attach` exempt | broken through the attach pre-flight (F4) |
| D7 refusal "by cwd" covers host-side lym over an area hub | broken: machine sweeps work by machine record, not cwd (F5) |
| D6 every right on the Go caches, with D13 deploy "with the host's toolchain", against the Goal's no persistence | broken (F6) |
| D8 bound: an agent cannot post as `EXTERNAL` to another hub | broken in two ways (F7) |
| D5 "never a bus": derived creel sockets come from machine-record entries | broken: the record is writable outside the area (F8) |
| D9 bound: the pid must descend from the supervisor | broken: the supervisor's own host-side children qualify (F9) |
| D9 "no child escapes the scope", parity with today | partly broken: the scope kill needs `/sys` write (F11) |
| D12 bound: the relay reads only the slack module and its own state | broken: its own state lives in the agent-writable hub (F10) |
| D4 "neither exposure uses a datagram" | held for the bus and tmux |
| D4 "loopback/TCP allowed" against the Goal "cannot act outside the hub" | broken as stated: loopback services are not mediated (F12) |
| D3 bound: `boot` takes no argument | held |
| D8 SCM_CREDENTIALS cannot be forged | held: probe table row "peer as the test socket's server sees it" |
| D13 bound: nothing deploys without the operator's command | held. "The commit is on that branch" adds nothing, since D11's token can push there; this is already stated in Rejected alternatives |
| D2: the start refuses a write list over host inputs | partly: the derived grants (hub root, Go caches) are not checked (folded into F6) |
| D15 profile named by a hash of its rendered text | held |

**Findings**

F1 — BLOCKING, design, decision-record.md D4/D1. An agent inside can install its own seccomp filter with `NEW_LISTENER` that returns USER_NOTIF on `connect`, and have a helper process answer `SECCOMP_USER_NOTIF_FLAG_CONTINUE`. The newest filter is evaluated first, and the first-seen action of equal precedence wins, so only the agent's listener is notified. The connect to `/run/user/<uid>/bus` then runs, Landlock ABI 8 does not mediate it, and the outer supervisor never sees it. `no_new_privs` is already set, so installing the filter needs no privilege. The probe's filter allows `seccomp` and `prctl`.
- Evidence: seccomp_unotify(2) NOTES: "a user-space notifier can be bypassed if the existing filters allow the use of seccomp(2) or prctl(2)…".
- Evidence: seccomp(2): "most recently installed filter is executed first … first-seen action value of highest precedence".
- Evidence: docs/research/platform/seccomp-connect-probe/main.go:677-689, where everything but connect and io_uring_setup is allowed.
- No probe row tests this.
- Fix: the filter refuses `seccomp(SECCOMP_SET_MODE_FILTER)` and `prctl(PR_SET_SECCOMP)`, or at least the `NEW_LISTENER` flag, and checks the cost to Codex's own seccomp sandbox. Add an acceptance criterion for a nested listener's CONTINUE.

F2 — BLOCKING, design, D6 attach client against Constraints (Yama). The attach client's ancestry is the operator's shell, not the supervisor. Under `ptrace_scope` 1 the supervisor can neither read `/proc/<tid>/mem` nor `pidfd_getfd` the client, so the client's connect to the tmux socket answers `EPERM` and the attach fails. The record's own Constraint says the supervisor reads memory "only as its subreaper (measured: EPERM without)". The probe hits the same case at main.go:316-317.

F3 — BLOCKING, design, D3/D6 area root. Agents hold every right on `/tmp/lyma-<uid>-<hash>`, which holds the supervisor's and tmux's sockets, and its name is predictable.
- An agent can unlink the supervisor socket and bind its own. A host-side `lym reed attach` then hands its filter listener to the agent, who answers CONTINUE for the client's connects. A `detach-client -E` command then reaches the bus.
- A host-side `lym area start` that asks `boot` talks to the same impostor.
- The record does not say who creates the root, checks its owner, or authenticates the peer (for example, by checking that SO_PEERCRED is the recorded supervisor pid).

F4 — BLOCKING, design, D7 exemption of `lym reed attach`. Host-side attach runs, before any restriction:
- `CheckTmuxFloor(c.eng.TmuxPath())`, which executes tmux;
- `EnsureSession`;
- `ensureWatchdogSpawned` with `TmuxPath()` and `ShellPath()`;
- `handOver(TmuxPath())`.

The tmux path comes from the hub's `reed.yaml` `tmux:` key. Agents write that file, so the host executes a binary the agent chose. D3's "never from hub config" covers only the supervisor's argv.
- Evidence: internal/reedcli/attach.go:56-105.
- Evidence: internal/reedcli/spawnwatchdog.go:12.
- Evidence: internal/reedengine/config.go:17.
- Evidence: internal/reedengine/template_posix.yaml:1.

F5 — MEDIUM, design, D7. The refusal keys on the hub that cwd resolves to. Host-side lym run from another hub still acts on an area hub's agent-written state. `lym machine start` and the sweeps walk every hub in the machine record, and creel's outbound worker builds other hubs' geometry. So host lym reads area-written config and runs git there.
- Evidence: internal/machinecli/hubs.go:74 (`hubsToSweep`).
- Evidence: internal/machinecli/cli.go:151.
- Evidence: internal/creelcli/hubdirectory.go:72.

F6 — BLOCKING, design, D6/D13/D2. D6 grants every right on "the Go build and module caches". These are by default the host's `~/.cache/go-build` and `~/go/pkg/mod`, which D2's start check does not list as host inputs.
- An agent can overwrite a build-cache output entry, or a dependency source in the module cache. Landlock has no `chmod` right, so the read-only mode does not stop it.
- `lym area deploy` builds "with the host's toolchain" (`go run ./tools/deploy` installs lym), and other hubs' host-side builds read the same caches. Either way agent-planted code is compiled into host binaries.
- That is persistence outside the area, against the Goal and D13's bound.
- Fix: give the area its own GOCACHE and GOMODCACHE, and add them to D2's fenced inputs.
- Evidence: tools/deploy/main.go:1-20; docs/research/platform/landlock-area-2026-11-23.md, allow-list rows for `~/.cache/go-build` and `~/go/pkg/mod`.

F7 — BLOCKING, design, D8/D3 inside test.
- (a) The writer of the request can exit before the daemon checks it, while another process holds the socket open. SCM_CREDENTIALS then names a dead pid, the ancestry walk finds nothing, and the peer is judged not inside. `resolveSender` then gives `EXTERNAL`, because a process in no pane is `EXTERNAL`.
- (b) An agent can make the operator's area-run attach client run `detach-client -E "lym creel post …"`. That process carries a filter, but its ancestry is the operator's shell, so it is not "inside". Its connect is allowed, because the other hub's creel socket is derived.
- In both cases the agent posts as `EXTERNAL` to another hub, which D8 calls an act outside the area.
- The decision must fail closed on a dead or unreadable pid (or use SCM_PIDFD), and must judge "carries a filter" without relying on ancestry.
- Evidence: internal/creelengine/server.go:246-253; internal/creelengine/identity.go:152-168; internal/creelengine/doc.go:48.

F8 — MEDIUM, design, D5/D6. Derived creel socket paths come from machine-record entries, "read at each decision". The record (`hubs.json` in the per-user state dir) is written by every non-area daemon and is writable by any process outside an area. An entry whose socket path is `/run/user/<uid>/bus` and which carries the area-aware level makes the supervisor allow the bus, against "never a bus". D5 states no check that a derived path is a creel socket by name and location.
- Evidence: internal/hubregistry/hubregistry.go:37-60.

F9 — MEDIUM, design, D9 bound. "The pid descends from it" also admits the supervisor's own host-side children: the D12 relay, Konsole and attach windows from D13, and possibly the supervisor itself. An area agent can `cap-memory` the relay or the supervisor with a tiny limit. That kills or confines host-side processes, and killing the supervisor fails every connect closed. The check should be D3's inside test, not descent.

F10 — MEDIUM, design, D12 bound. The relay holds the operator's Slack tokens, and its "own state" sits in agent-writable paths: the setup file's `operator_id`, the channel map, the bridge map, the outbox and the forwarding file.
- An agent can name another Slack user as the operator, or queue posts through the operator's bot.
- So a host process with outside secrets acts on area-written input, and the stated bound does not cover it.
- Evidence: internal/slackengine/geometry.go:75-103; internal/slackengine/credentials.go:41-43; internal/slackengine/start.go:175-195.

F11 — LOW, design, D9. `RunMemoryCapped` ends a killed or OOM-killed scope through `killScope`, which writes the scope cgroup's `cgroup.kill` under `/sys/fs/cgroup`. D6 grants `/sys` read only, so the tree kill fails inside the area and survivors outlive the cap, unlike today.
- Evidence: internal/proc/memorycap_linux.go:198-203, 249.

F12 — MEDIUM, design, D4/D16 against the Goal. D4 allows every TCP and UDP connect, loopback included. Any host service on 127.0.0.1 or ::1, such as a debug port, a local dev server or an editor bridge, is reachable from inside without mediation. D16's non-claims name only "the open network" and datagram sends, so the Goal's "cannot act outside the hub" overclaims. Either bound loopback in the supervisor or name it as a non-claim.
- Evidence: decision-record.md D4 and D16; seccomp-connect-probe/main.go:399-401.

Tool calls:
1. seccomp_unotify(2) CONTINUE text and seccomp(2) precedence: the nested-filter semantics were not in context.
2. seccomp_unotify(2) NOTES and seccomp(2) filter ordering: the exact bypass wording.
3. tools/deploy header: what `lym area deploy`'s build runs on the host.
4. reed `Tmux` uses and the attach watchdog spawn: whether the attach pre-flight runs a tmux from hub config.
5. reed config field and templates: confirmed `tmux:` is a hub `reed.yaml` key.
6. slackengine geometry and the Setup struct: the relay's agent-writable inputs.
