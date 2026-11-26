# A closed area for lym: Landlock and a seccomp connect supervisor per hub

## Goal

Give a hub an area the operator turns on, in which every agent keeps bypass permissions
and cannot act outside the hub, read a secret outside it, or persist code that later runs outside it.
Two exposures hold today with no area:
an agent reaches `/run/user/<uid>/bus` and through it `systemd-run --user`, which starts any process outside lym's tree,
and it types into another hub's tmux panes through their sockets.
The area closes both on kernel 7.0 with no root and no user namespace,
following the ranking of `docs/research/platform/sandbox-without-new-kernel-2026-11-23.md` (`57d15cc02`):
a seccomp filter that hands every `connect()` to a supervisor in lym, stacked over Landlock ABI 8, first;
an AppArmor profile per hub as the fallback.
The area keeps the strand pane memory cap of `10ae7e112`, whose scope is made over the session bus the area refuses.
With the area off, everything works as it does today; with it on, nothing starts when the area cannot.

## Scope

In:

- The area's mechanism, boot, allow list, Landlock ruleset and per-hub roots (D1 to D6).
- What changes for host-side lym, creel, the memory caps, engine config directories, the GitHub token and the Slack relay (D7 to D12).
- The host exits the supervisor serves (D13), the build order with its stops (D14), the AppArmor fallback (D15) and the docs (D16).
- What stays of the entry's three `bwrap` stages: stage 1's `bwrap` area is replaced by D1;
  stage 2's egress proxy is dropped and its GitHub token kept (D11); stage 3's broker becomes D13's supervisor verbs.

Out:

- The shellguard PreToolUse rows for the bus commands and other hubs' tmux servers: task `reed-client-and-bus-guard` owns them,
  and the area relies on none of them.
- Claude Code deny rules in shuttle's settings as a second line.
- Network egress control, and every other non-claim D16 lists.

## Decisions

### D1 — One Landlock domain and one seccomp connect filter per hub, no bwrap

The area is the hub's tmux server and every process under it, restricted by a Landlock ABI 8 ruleset (D6)
and by a seccomp filter that hands every `connect` to the hub's supervisor (D4).
No `bwrap`, user namespace, AppArmor profile or root step is part of it on the seccomp route.
New packages `internal/areaengine` (new) and `internal/areacli` (new) own it, wired as `lym area` in `cmd/lym/main.go`.
Rationale: the study measured this route closing both exposures on Neve with no root;
`bwrap` needs an AppArmor profile OSL-1046 cannot load without IT, and Landlock ABI 8 alone closes neither exposure (measured).

### D2 — Off by default; a per-hub area file outside the hub turns it on

A hub's area is on when its area file `~/.config/lym/area-<hub key>.yaml` exists, `<hub key>` being reed's server name for the hub.
It holds `read`, `write` and `exec` path lists, `connect` socket paths for the allow list,
the barrier (`seccomp`, the default, or `apparmor` per D15) and an optional `tmux` path.
Only `internal/areaengine` (new) reads it, from outside the area; it is no `configengine` module, and no hub layer holds an area setting.
`lym area init` writes it in full the first time, with the Landlock study's read list as its content.
The start refuses a file whose `write` list grants a path on, under or above a host-side input:
the area file and token file (D11), the area state directory (D3), `~/.claude`, `~/.codex`, the installed `lym` and the Go toolchain.
Bound: an agent cannot turn the area off or widen it, since every hub layer is writable inside and none of those inputs is.
Rationale: a setting in a hub layer would let an agent disable its own area at the next start.

### D3 — Boot, lifetime and the inside test

- The area state directory, `<per-user lym state dir>/area/<hub key>/`, holds the supervisor record (its pid and the barrier) and nothing the area writes;
  D6 grants read on the record alone.
- `lym area start`, from a host shell, checks the kernel and the file and, with no live supervisor, spawns a detached `lym area supervise`.
  The check refuses, naming what is missing: Landlock ABI below 8 (no `TSYNC`), no seccomp user notification, no `pidfd_getfd`, or a file failing D2.
  With a live supervisor and no server, it asks the supervisor's `boot` verb instead.
- `lym area supervise` sets itself as child subreaper, writes the record, opens its socket and verbs (D9, D12, D13), and boots the server.
- `boot` starts `lym area exec -- <tmux argv>` over an inherited socket pair when no server runs, and does nothing otherwise.
  The supervisor builds the argv from `PATH` or the file's `tmux`, never from hub config; the hub's `tmux_conf` and `shell` apply inside the server only.
- `lym area exec` applies the Landlock ruleset with `TSYNC`, sets `no_new_privs`, installs the filter with `TSYNC` and `NEW_LISTENER`,
  sends the listener to the supervisor and `exec`s tmux, so the server and every pane carry both.
- reed's server spawn in `internal/reedengine/lifecycle.go`, for a hub whose area is on, goes through `lym area start` from outside and through `boot` from inside.
- The inside test: a process is inside a hub's area when it carries the hub's barrier, a seccomp filter (`Seccomp_filters` in `/proc/<pid>/status`) for `seccomp`
  or the hub's profile label (`/proc/<pid>/attr/current`) for `apparmor`, and its ancestry reaches the recorded supervisor.
- The supervisor stays while any process of its tree lives, so it decides the connects of dobby and other orphans after the server exits.
  `lym area stop` ends the hub's runs and hub processes through `runproc`'s end verbs, ends every process left in the supervisor's tree with `runproc.EndProcess`, then the supervisor.
- dobby and the creel daemon join the hub's one domain because a process inside starts them;
  their spawn sites refuse from outside an area that is on, naming the way forward: start them from a pane of the hub's server.
  A hub without an area never starts an area hub's creel daemon; its outbound worker retries.
Bound: `boot` takes no argument, so a caller inside can only restart the server the supervisor would start.
Rationale: Landlock's signal and abstract-socket scopes admit only one domain or its children, so the tree must start from one restriction.

### D4 — The supervisor decides every connect, and fails closed

The filter refuses every system call of a foreign ABI, i386 and x32, with `ENOSYS`, refuses `io_uring_setup` with `EPERM`,
hands `connect` to the supervisor and allows the rest.
For each `connect` the supervisor runs the algorithm of `docs/research/platform/seccomp-connect-probe/main.go`:
it reads the address through `/proc/<tid>/mem` and checks the notification is still valid;
refuses every abstract Unix address;
opens a pathname with `O_PATH`, through `/proc/<tid>/cwd` when relative, and refuses a canonical path off the allow list with `EACCES`;
takes the agent's socket with `pidfd_getfd` and connects it itself, a Unix one to `/proc/self/fd/<the O_PATH descriptor>`;
and answers the notification with the connect's result.
TCP and UDP connects are allowed and made by the supervisor too, since the family it checked could change before the connect.
The supervisor also takes listeners handed to it over its socket (D6's attach client).
Bound: the filter covers `connect` alone; a `sendto` or `sendmsg` to a named datagram socket, such as the journal's, passes,
and neither exposure uses one.
A supervisor that dies leaves every connect in the tree failing with `ENOSYS`; the filter cannot be handed to a new listener,
so the hub restarts with `lym area stop` and `lym area start`.
Rationale: making the connect on the checked inode closes the address race (measured 0 of 200 reaching the bus, against 42 unfiltered).

### D5 — The allow list: derived sockets plus the area file, never a bus

Derived entries: the hub's own tmux socket, its own creel socket and the supervisor's socket,
and the creel socket of each other hub whose machine-record entry carries the area-aware creel level of D8, read at each decision.
The area file's `connect` list adds more.
The session bus, `/run/user/<uid>/systemd/private`, the system bus, the SSH agent, the desktop's and VS Code's sockets,
and other hubs' tmux sockets are on no derived list.
`lym area show` prints the kernel's Landlock ABI, every allowed socket and every Landlock path with its rights and its source, derived or file,
and the supervisor logs the same list at its start.
Bound: a socket the operator adds to the file is reachable by every agent of the hub.
Rationale: the bus is the `systemd-run` escape, so only a supervisor verb reaches it, never an agent's connect.

### D6 — The Landlock ruleset, the area root and the attach client

- Read and execute: the system directories, the Go toolchain, the Claude Code launcher and versions, the installed `lym`, `~/.gitconfig`, `~/.config/git`
  and `~/.config/lym/machine.json`, as the Landlock study measured;
  read: `/proc`, `/sys`, `/dev`, `/run`, the machine's hub record, the supervisor record, the token file (D11) and the host's Claude plugin cache.
- Every right: the hub root, which holds the area's engine config directories (D10), the Go build and module caches,
  and the area root `/tmp/lyma-<uid>-<8 hex of the hub path's hash>`; `/dev/null`, `/dev/pts`, and write and ioctl on `/dev/ptmx`.
- `/tmp`, `$HOME` and `~/.config` get no write.
- Inside the area, `TMPDIR`, `GOTMPDIR` and `TMUX_TMPDIR` point at the area root, so lym's temp root (`internal/lymtemp/doc.go`),
  the hub's creel socket, its tmux socket and the supervisor's socket live under it, and other hubs' temp roots and sockets stay unwritable.
- Each hub's daemon writes its creel socket path into its machine-record entry, and `lym area start` writes it for an area hub;
  creel finds another hub's socket from that entry, never from the caller's own temp root.
  A host-side `lym reed attach` finds an area hub's tmux socket from the area root, which it derives from the area file.
- The operator's tmux client of an area hub runs under the area too: `lym reed attach` applies the hub's ruleset and a filter whose listener it hands to the supervisor, then `exec`s the client.
  So a `detach-client -E` command an agent sends runs inside the area, and the client's signal to its parent shell is refused by Landlock's signal scope.
Bound: the ruleset only adds access, so a path it leaves out, `~/.ssh` and `~/.config/gh` among them, is denied.
Rationale: a write grant on the shared `/tmp/lym-<uid>` or `/tmp/tmux-<uid>` would let an agent rewrite another hub's run directory or socket,
and a host tmux client runs whatever command the server tells it to replace itself with.

### D7 — Host-side lym refuses over an area hub

With a hub's area on, every `lym` verb whose cwd resolves to that hub and that runs outside the area refuses,
bar `lym area`, `lym reed attach`, the supervisor-started relay of D12, `lym version` and help,
naming the way forward: run it from a shell window of the hub's server.
The check is the first step of the root pre-run in `cmd/lym/main.go`, ahead of every other root-level step, and an exempt verb skips those steps' hub reads.
Bound: the refusal covers lym only; the operator's own `git` or VS Code over the hub is outside lym's reach, which D16's docs state.
Rationale: lym on the host reading agent-written hub config, or running git over a repository whose `.git/config` and hooks agents write,
would run agent-chosen code outside the area.

### D8 — creel across an area

- Every daemon sets `SO_PASSCRED` and takes the sender's pid from `SCM_CREDENTIALS`, which the kernel attaches to each message,
  since `SO_PEERCRED` names the supervisor for every connect it makes (measured).
  `peerPID` in `internal/creelengine/peer_linux.go` changes accordingly.
- Within a hub, senders resolve as today: a pane's strand, a program, or `EXTERNAL` for a process in no pane.
- Every daemon serves a peer inside another hub's area, by D3's inside test on the peer's pid, only the cross-hub verbs `deliver` and `confirm`;
  every other verb refuses, so an agent cannot post as `EXTERNAL` to another hub.
- A daemon that does this records the area-aware creel level in its machine-record entry; D5 admits only hubs whose entry carries it,
  so a daemon of an older binary is never reachable from an area.
- The machine record is written for an area hub by `lym area start`, and read-only inside; the in-area daemon skips its own record write.
- Inside an area, the outbound worker never starts another hub's daemon; it retries until that hub starts its own.
Bound: the kernel refuses an unprivileged sender's `SCM_CREDENTIALS` naming any pid but its own.
Rationale: `EXTERNAL` may inject and interrupt any strand of a hub, so reaching another hub's daemon as `EXTERNAL` is an act outside the area.

### D9 — The memory caps go through a supervisor verb

Inside an area, `proc.CapProcessMemory` and `proc.RunMemoryCapped` ask the supervisor's `cap-memory` verb in place of `busctl` or `systemd-run`.
The verb takes a pid, a limit and a kind, pane or run, and makes the transient scope `internal/reedengine/panememory.go` and `internal/proc/memorycap_linux.go` make today,
under the kind's own unit prefix, with `MemoryMax`, no swap and OOM policy `continue`.
`RunMemoryCapped` starts a held child that waits on a pipe, has the supervisor cap it, and only then releases it to `exec` the command,
so no child escapes the scope; a refused cap falls back to `RLIMIT_DATA` as today.
Bound: the supervisor opens a pidfd first and then checks that the pid descends from it; it sets only those properties and starts no process,
so an agent gains no bus call through it.
The OOM watch reads its cgroup as today; the journal is unreadable inside, so its notice falls back to a count.
Rationale: the operator requires the pane cap to survive an area that refuses the bus.

### D10 — The area's own engine config directories

Inside an area, every engine's config directory lives under `<hub>/.lym/area/`, one per engine and per account:
claudeengine's account resolution maps each account, the default one included, to its directory there and ignores the env module's `CLAUDE_CONFIG_DIR`;
the launch line sets `CLAUDE_CONFIG_DIR` and `CLAUDE_CODE_TMPDIR`, the latter at the area root, and codexengine's sets `CODEX_HOME`.
Every lym reader of an engine's config directory, `TranscriptReader` and the user-setup source in `internal/shuttleengine/claudeengine` among them, resolves the same directory.
The host's plugin cache is granted read-only and linked into the area's Claude config directory.
Each engine is logged in once inside the area.
Bound: the area's config directories are writable to its agents, so their hooks run only in the area's own sessions.
Rationale: writing the host's config directory is the persistence exit, and `PATTERN-engine-parity` holds every engine to the same area.

### D11 — The area's own GitHub token

Inside an area, `internal/githubclient` reads only the area's token file, `~/.config/lym/area-<hub key>.token`, granted read-only,
and git's HTTPS credential comes from `lym area git-credential`, which reads the same file:
the area's launch environment sets git's credential helper for the hub's host to it through `GIT_CONFIG_COUNT`, after an empty value that clears the host's helpers.
The operator's `gh` login, `~/.config/gh` and lym's credential directory stay denied.
The operator writes a fine-grained token limited to the hub's repositories there.
Bound: the token grants what GitHub grants it, the push and pull request reach a run already has.
Rationale: stage 2's token rule keeps the operator's own GitHub reach out of the area.

### D12 — The Slack relay runs outside the area, owned by the supervisor

Inside an area, `slackengine.EnsureRelay`, its away variant and every path that ends the relay ask the supervisor's `slack-relay` verb, `start` or `stop`.
The supervisor starts the relay as its own host-side child and ends it at `stop` or `lym area stop`; it holds no `runproc` record an area process would try to signal.
The relay reads its Slack tokens from lym's credential directory, which the area cannot read, and D7 exempts it.
Bound: the relay reads only the hub's slack module, whose template holds one key, `remind_after_min`, and no executable (`internal/slackengine/template.yaml`), and its own state.
Rationale: the relay's tokens are a secret outside the area.

### D13 — Host exits as fixed supervisor verbs

The supervisor serves, besides D3, D9 and D12, verbs that take no command line from the area:

- `window`: a worktree under the hub, checked by canonical path, and a title; it runs Konsole with the host's `lym reed attach`, which runs its client under the area (D6).
  `lym reed window` in `internal/termwindow/termwindow.go` routes to it inside an area; no hub config key, the post-window command included, applies to it.
- `hibernate`: `lym hibernate request` pauses every session as today and then asks the verb, which runs `systemctl hibernate` and nothing else.
- `deploy`: Batten's Land row (`internal/battenshed/land.go`) records a pending deploy of the landed commit through the verb and goes on, with a notice naming `lym area deploy`.
  The operator runs `lym area deploy` from a host shell; it shows the pending commit, fetches the parent branch from origin into its own clone in the area state directory,
  checks the commit is on that branch, and runs the area file's deploy command there with the host's toolchain.
- `lym ide` refuses inside an area, naming the reason: VS Code on the host runs git over repositories the area writes.
Bound: no verb runs area-chosen code on the host; nothing deploys without the operator's command.
Any area process can hibernate the host or open an attach window; neither runs area code outside the area.
Rationale: these are the bwrap design's stage 3 exits, still broken inside this area, and a deploy on request would run any commit an agent pushed.

### D14 — Build order with two stops

1. Before any task: the operator runs `docs/research/platform/seccomp-connect-probe/main.go` on OSL-1046, under Cortex XDR.
   Its result picks OSL-1046's barrier: `seccomp`, or D15's `apparmor` when the probe fails there.
2. Stage 1: D8's creel identity first, landed before the boot; then D1 to D12 and D16.
3. Stop: the operator runs the area on both machines and decides whether it works well enough to go on.
4. Stage 2: D13's verbs; until then the operator opens windows, hibernates and deploys by hand from a host shell.
5. D15 is built only when step 1 picks `apparmor`.
The note is split into tasks or a project only when the operator decides it goes ahead.
Rationale: the operator doubts the whole works without large problems, and the study puts the OSL-1046 probe ahead of the first task.

### D15 — The AppArmor profile per hub as the fallback barrier

With `barrier: apparmor` in the area file, `lym area profile` renders a profile for the hub from the same allow list and Landlock paths,
denying every Unix-socket connect off the allow list, named with a hash of its rendered text, and names the root command that loads it.
`lym area start` enters it with `aa-exec -p <profile>` in place of the seccomp filter, and the attach client of D6 enters it too;
Landlock still applies, and the supervisor still serves D3's `boot`, D9, D12 and D13.
The start refuses when no loaded profile in `/sys/kernel/security/apparmor/profiles` carries the name the current file and geometry render.
Bound: unmeasured on both machines; each new hub or changed list is one more root load, by IT on OSL-1046.
Rationale: the study's ranking keeps it for the case the supervisor fails on OSL-1046 and IT will load profiles.

### D16 — The docs and the repository's claim

`README.md` states that every agent lym starts runs with bypass permissions, that lym belongs on a machine the operator will hand to its agents,
and that a hub without an area is best booted from a Konsole window, not a VS Code terminal, whose AppArmor label grants agents user namespaces.
For an area hub it states what the area closes and what it does not claim:
a kernel exploit; data leaving over the open network or a datagram send; landed lym code running on the host once deployed;
which sender posted within the hub, since an agent can still post as a program or `EXTERNAL` there as today;
and what pane output and clipboard writes reach the operator's terminal.
It states that with an area on, the operator runs lym and git over the hub only from a shell window of the hub's server, and opens no area worktree in host VS Code.
`docs/overview.md` describes `lym area`, the area file and the supervisor verbs; the new packages' `doc.go` files state their bounds.
The glossary gains `area`.
Glossary line: **area**: a hub's closed area: one Landlock domain and connect barrier around its tmux server, its supervisor outside. Never: jail, container.
Rationale: the claim is what the operator wanted the repository to be able to state.

## Constraints

- Kernel facts, from `docs/research/platform/sandbox-without-new-kernel-2026-11-23.md` and its man-page sources:
  seccomp user notification is Linux 5.0, `pidfd_getfd` 5.6, `TSYNC_ESRCH` 5.7, `PIDFD_THREAD` 6.9;
  Landlock ABI 8 with `LANDLOCK_RESTRICT_SELF_TSYNC` measured on Neve's 7.0.0-31; OSL-1046 runs 7.0.0-42, ABI 8 unmeasured.
- Yama's `ptrace_scope` is 1 on both machines' Ubuntu default, so the supervisor reads an orphan's memory only as its subreaper (measured: `EPERM` without, ok with).
- lym is built with cgo (`tools/deploy` pins `CGO_ENABLED=1`), under which `syscall.AllThreadsSyscall` refuses; both restrictions use `TSYNC` for that reason.
  The probe set `no_new_privs` with an all-threads call, so `TSYNC` copying it to every thread is unmeasured in lym's cgo build.
- tmux tries the session bus to put each pane in a scope; refused, it keeps the pane in the server's cgroup (measured).
- tmux's `detach-client -E` replaces a client with a shell command, and `-P` signals the client's parent (tmux man page, `detach-client`).
- The pane cap is made by the lym process that launches the pane, through `busctl`, with an uncapped fallback and a Warn;
  `RunMemoryCapped` uses `systemd-run --user --scope` with an `RLIMIT_DATA` fallback.
- Claude Code reads `CLAUDE_CONFIG_DIR` and `CLAUDE_CODE_TMPDIR`: both names are in the installed binary's strings on Neve.
  reed's server spawn scrubs both from its environment, so they reach a pane only through the launch line.
- `PATTERN-shell-guard-table`: the area adds no shellguard row.

## Auto-mode assumptions

- The entry's "config module `area`" becomes a file outside the hub (D2).
- Network egress stays open; stage 2 keeps only its token rule (D11).
- The system bus stays off the allow list (D5).
- Within a hub, creel's sender rules stay as today (D8).
- The landing deploy needs the operator's host-side command (D13).
- The AppArmor fallback is built only when the OSL-1046 probe fails (D14).

## Open risks

- The supervisor probe has not run on OSL-1046; whether Cortex XDR interferes with a seccomp listener or `pidfd_getfd` is unknown.
  D14 makes that run the first step, and its result picks the barrier.
- How another hub's daemon confirms a sender whose connect the supervisor made is unmeasured; D8's `SCM_CREDENTIALS` is the design's answer.
- A glibc program asking `systemd-resolved` over its Unix socket is refused and falls back to DNS, unmeasured; the area file can add the socket.

## Acceptance criteria

- D1, D3: on a kernel with Landlock ABI 8, `lym area start` boots the hub's tmux server so that a pane process carries both the Landlock domain and the seccomp filter, and `lym area exec` run alone refuses without a supervisor's socket pair.
- D3: `lym area start` refuses, naming the missing feature, when the kernel lacks Landlock ABI 8, seccomp user notification or `pidfd_getfd`; with a live supervisor and no server it boots a new server under that supervisor.
- D3: reed boots an area hub's server through `lym area start` from outside and through `boot` from inside; dobby and creel spawn sites refuse from outside with their way forward.
- D3: an orphan of a pane, double-forked, still gets its connects decided after reparenting; `lym area stop` leaves no process of the supervisor's tree alive.
- D3, D15: the inside test holds for a pane process under each barrier, and fails for a host process and for the supervisor's host-side children.
- D4: under the filter, a connect to the session bus, to a symlink to it, to another hub's tmux socket and to an abstract address fails with `EACCES`, and the hub's own tmux and creel sockets connect.
- D4: 200 connects from a buffer another thread swaps between an allowed socket and the bus never reach the bus.
- D4: a foreign-ABI system call fails with `ENOSYS`, `io_uring_setup` with `EPERM`, and every connect fails with `ENOSYS` once the supervisor's listener is closed.
- D2, D5: with no area file the hub boots as today; `lym area show` lists every derived and file entry with its source; a file whose `write` list covers `~`, `~/.config` or the installed `lym` is refused at start.
- D5: a socket the area file adds to `connect` is reachable from a pane; the session bus and the system bus are not; another hub's creel socket is reachable only while its machine-record entry carries the area-aware level.
- D6: inside the area, a write under `$HOME`, `/tmp` outside the area root, another hub's lym temp root and `~/.config/lym` fails, and the hub, the Go caches and the area root are writable.
- D6: a host-side `lym reed attach` of an area hub finds its server, and a `detach-client -E` sent to that client runs its command under the area's filter.
- D6, D8: a cross-hub post between an area hub and a hub without one is delivered and confirmed in both directions, each side finding the other's socket from the machine record.
- D7: a lym verb run outside an area hub's area refuses with the way forward before any other root step runs, and `lym area show` and `lym reed attach` run.
- D8: a post from a pane of an area hub is attributed to the pane's strand; a daemon refuses every verb but `deliver` and `confirm` from a peer inside another hub's area.
- D8: inside an area the outbound worker does not start another hub's daemon, and a hub without an area does not start an area hub's daemon.
- D9: a strand pane launched in an area lands in its pane scope with its `MemoryMax`; a `RunMemoryCapped` child's own children land in the run scope; the verb refuses a pid outside the supervisor's tree.
- D10: an agent session in an area, under a listed account and under the default one, writes its transcript under `<hub>/.lym/area/`, and `TranscriptReader` reads it there; the host's `~/.claude` is unwritable inside.
- D11: inside an area, `githubclient` resolves the area's token file and never `gh auth token`, and a `git push` over HTTPS authenticates through `lym area git-credential`.
- D12: the relay started through `slack-relay start` from inside runs as the supervisor's child outside the area, and `slack-relay stop` ends it.
- D13: the `window` verb refuses a worktree outside the hub; the Land row's deploy inside an area records a pending deploy and runs nothing on the host; `lym area deploy` refuses a commit not on origin's parent branch; `lym ide` inside an area refuses with its reason.
- D13: `lym hibernate request` inside an area reaches `systemctl hibernate` only through the `hibernate` verb.
- D15: with `barrier: apparmor`, `lym area profile` renders a profile denying the session bus and other hubs' sockets, and `lym area start` refuses while no loaded profile carries its rendered name.
- Review: D14 — the note's split into tasks follows D14's order, with D8's creel identity ahead of the boot.
- Review: D16 — `README.md` states bypass permissions, the Konsole advice, the area's claim, each non-claim and the operator's host-side rules; `docs/overview.md` and the glossary carry `area`.

## PATTERN changes

- `PATTERN-closed-area` — adds: Running a hub in an area: booted only via its supervisor; one Landlock ruleset and connect barrier per hub; only the supervisor decides a connect, by derived paths and the area file only `areaengine` reads; a host exit only as a supervisor verb, never an area's command line.
- `PATTERN-child-processes` — changes: Starting or ending a process: logged via `internal/logger`; its starter waits on it, bar a double fork's middle; a run's processes register in and end only via `internal/runproc`, orphans via `EndProcess`; lym named via `selfexec.LymBinary`; only an area supervisor subreaps.

## Notes for the plan writer

- The probe's supervisor (`docs/research/platform/seccomp-connect-probe/main.go`) is the starting point for D4; it sits on the security boundary, and its tests run on a real kernel at the lowest tier that runs one.
- `PATTERN-config-layers` keeps its line: the area file is no config module; its background gains one sentence naming the area file as outside its layers, merged with the line `config-keys-nested` rewrites.
- `golang.org/x/sys/unix` is already a direct dependency and carries the Landlock types the probes use.
