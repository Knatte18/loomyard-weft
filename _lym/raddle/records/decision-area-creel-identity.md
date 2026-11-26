## Goal

creel across an area: kernel-attached sender identity, the area registry and the hub directory

Takes: the Sketch's decision record, D8 whole and the pieces of D3 and D6 that D8 rests on; peerPID (internal/creelengine/peer_linux.go), daemon.resolve and resolveSender, serveHubCall (receive.go), the outbound worker's spawn closure (outbound.go attempt), hubregistry's Entry, Lookup, Live and RepositoryName, creelcli's hubdirectory and its one machine-record write
Delivers: every creel daemon sets SO_PASSCRED and SO_PASSPIDFD and takes the sender from SCM_CREDENTIALS and SCM_PIDFD with the pidfd liveness check; D3's inside test (barrier in /proc/<pid>/status or attr/current, plus a cgroup naming a scope of the hub's area by the whole-name forms lym-area-<hub key>.scope and lym-area-<hub key>-<pane|run|attach>-<pid>.scope), with which a daemon serves only deliver and confirm to a peer inside another hub's area or a peer gone or unreadable; the machine record moved to the hub directory <per-user lym state dir>/hubs/ with its lock, read under a shared lock on a read-only descriptor, with the old hubs.json read for entries the new record lacks; each daemon writing its shortname, creel socket path, repository name and the area-aware creel level into its entry, bar an in-area daemon; the area registry area-hubs.json and its lock in the hub directory, its reader, and a writer the core part's lym area start calls; hubregistry.Lookup, the confirm's geometry and RepositoryName reading an area hub's registry entry and, inside an area, every other hub's host-written record, never a Bolt shortname file or a prime's .git/config; every registry reader and writer dropping an entry whose area file ~/.config/lym/area-<hub key>.yaml is gone (Added after Discussion); the outbound worker starting no other hub's daemon inside an area, and a hub without an area starting no area hub's daemon; PATTERN-agent-addressing's Across hubs background gaining the area-registry repository rule
Exports: areaengine.HubKey (reedengine.ServerName of the canonical hub path), areaengine.FilePath(hubKey), areaengine.ScopeName and the scope-name parser, areaengine.Inside(pid, hubKey, barrier); hubregistry.AreaEntry {hub_path, shortname, creel_socket, repository}, hubregistry.ReadAreaRegistry and hubregistry.WriteAreaEntry, the hub-directory path helper; the area-aware creel level constant D5 admits

## Scope

The part changes these packages:

- internal/creelengine (about 450 changed lines)
- internal/hubregistry (about 350 changed lines)
- internal/creelcli (about 150 changed lines)
- internal/areaengine (about 250 changed lines)
- internal/lymdirs (about 20 changed lines)
- internal/machinecli (about 30 changed lines)
- internal/slackcli (about 20 changed lines)

## Decisions

### D8 — creel across an area

- Every daemon sets `SO_PASSCRED` and `SO_PASSPIDFD` and takes the sender from `SCM_CREDENTIALS` and `SCM_PIDFD` (Linux 6.5),
  which the kernel attaches to each message, since `SO_PEERCRED` names the supervisor for every connect it makes (measured).
  After its `/proc` reads of the pid, the daemon checks that the pidfd's process still lives (its fdinfo `Pid:` is no `-1`),
  so a pid reused after the sender exits never stands for it.
  `peerPID` in `internal/creelengine/peer_linux.go` changes accordingly.
- Within a hub, senders resolve as today: a pane's strand, a program, or `EXTERNAL` for a process in no pane.
- Every daemon serves only the cross-hub verbs `deliver` and `confirm`, and refuses every other verb, to a peer
  that is inside another hub's area by D3's inside test, the operator's attach clients, their commands and every orphan of them included,
  or whose process is gone or unreadable.
  So an agent cannot post as `EXTERNAL` to another hub by exiting before the daemon reads it, through the operator's attach client or an orphan of its command,
  after its supervisor dies, or from a hub started after the judging one.
- The test reads the peer's `/proc` entries alone, which D6 grants read inside every area, and no supervisor record.
- A daemon receiving from an area hub takes that hub's shortname, socket path and repository name from host-written data alone:
  the area registry `area-hubs.json` in the hub directory (D6), one entry per area hub,
  which only `lym area start` writes and no machine-record write touches.
  `lym area init` writes the area file's `shortname`, read from the hub's shortname record before its area is first on,
  and refuses a hub with none, naming the way forward: run `lym creel inbox` in that hub once, then `lym area init` again.
  The operator writes its `repository`, the GitHub owner/repo the hub works on; a file with none leaves the hub's mail untrusted at every receiver.
  `lym area start` copies both, with the hub's creel socket path, into the hub's registry entry.
  A hub the registry lists is an area hub: `hubregistry.Lookup`, the confirm's geometry and `RepositoryName` read its registry entry,
  never its Bolt shortname file or its prime's `.git/config`.
- A daemon inside an area takes every other hub's path, shortname, repository name and socket path from host-written records alone,
  never from that hub's files: a hub without an area from its machine-record entry, which that hub's own daemon writes (D6),
  and an area hub from its area-registry entry.
  So inside an area, `hubregistry.Lookup` and `RepositoryName` read no hub's Bolt shortname file and no prime's `.git/config`.
- A daemon that does this records the area-aware creel level in its machine-record entry; D5 admits only hubs whose entry carries it,
  so a daemon of an older binary is never reachable from an area.
- The machine record is written for an area hub by `lym area start`, and read-only inside; the in-area daemon skips its own record write.
- Inside an area, the outbound worker never starts another hub's daemon; it retries until that hub starts its own.
Bound: the kernel refuses an unprivileged sender's `SCM_CREDENTIALS` naming any pid but its own.
An area hub's creel socket sits in its area root, so any agent of that area can bind its own listener there and answer `confirm` for that hub,
and any agent can already drive the orch's pane through the hub's own tmux socket;
a confirm from an area hub proves only that the mail came from inside that area, under the repository its area file names,
and another hub's trust in an area hub is trust in every agent of it (D16).
Rationale: `EXTERNAL` may inject and interrupt any strand of a hub, so reaching another hub's daemon as `EXTERNAL` is an act outside the area.

### D14 — Build order with two stops

1. Before any task: the operator runs `docs/research/platform/seccomp-connect-probe/main.go` on OSL-1046, under Cortex XDR.
   Its result picks OSL-1046's barrier: `seccomp`, or D15's `apparmor` when the probe fails there.
   The same run reads OSL-1046's `/etc/nsswitch.conf` `hosts:` line and `/etc/resolv.conf`, as the Constraints record Neve's.
2. Stage 1: D8's creel identity first, landed before the boot; then D1 to D12 and D16.
3. Stop: the operator runs the area on both machines and decides whether it works well enough to go on.
4. Stage 2: D13's verbs; until then the operator opens windows, hibernates and deploys by hand from a host shell.
5. D15 is built only when step 1 picks `apparmor`.
The note is split into tasks or a project only when the operator decides it goes ahead.
Rationale: the operator doubts the whole works without large problems, and the study puts the OSL-1046 probe ahead of the first task.

### Added after Discussion (parent, 2026-11-26): The shellguard bus and hub-tmux rows are built

Decision: Treat the study's first recommendation as built by 2ed54e596: the session-bus row refuses systemd-run, busctl --user, gdbus and dbus-send, and the hub-tmux-server row refuses tmux -L or -S aimed at a lym- server; both are guardrails, not barriers. The entry body's sentence that no bus row exists is out of date.

Rationale: Landed on main while this motif ran (board tidy 2026-11-26).

### Added after Discussion (arbiter, 2026-11-26): D3's filter install sets TSYNC_ESRCH

Decision: In D3's `lym area exec` bullet, replace "then installs the filter with `SECCOMP_FILTER_FLAG_TSYNC` and `NEW_LISTENER`," with "then installs the filter with `SECCOMP_FILTER_FLAG_TSYNC`, `TSYNC_ESRCH` and `NEW_LISTENER`,". D6's attach client installs its filter with the same three flags. D15's filters, which have no listener, set `TSYNC` and `TSYNC_ESRCH`.

Rationale: seccomp(2) refuses `TSYNC` together with `NEW_LISTENER` with `EINVAL` unless `TSYNC_ESRCH` is set. The probe D4 starts from sets all three (`docs/research/platform/seccomp-connect-probe/main.go:646`), and Constraints already lists `TSYNC_ESRCH` (Linux 5.7). The round 3 review held this back (LOW).

### Added after Discussion (arbiter, 2026-11-26): lym area deploy takes its origin and branch from the area file

Decision: D2's list of area-file keys gains `deploy`, the deploy command D13 runs, and `deploy_branch`, which `lym area init` writes as `main`. `lym area deploy` clones `https://github.com/<repository>.git`, `<repository>` being the area file's `repository` (D8), into its clone in the area state directory. It fetches `deploy_branch` and checks that the pending commit is on that branch. It takes neither the URL nor the branch from the hub's `.git/config`, hub config, Bolt or any hub record. If `repository`, `deploy` or `deploy_branch` is missing, it refuses with the way forward: write them into the area file from a host shell. New D13 acceptance criterion: with the prime's origin rewritten to another URL and a hub record naming another branch, `lym area deploy` fetches from the area file's repository and checks against its `deploy_branch`.

Rationale: D2's rationale makes the area file the only host-written input, since agents write every hub layer. Taken from hub records, an agent would choose both the URL and the branch, and the on-branch check would prove nothing. The board entry's stage 3 fetches `origin/main`, and the operator already writes `repository` into the area file (D8). The round 3 review held this back and rated it BLOCKING in weight.

### Added after Discussion (parent, 2026-11-26): Pseudo-terminals are a stated bound; TIOCGPTPEER panes are an open risk

Decision: D6's Bound and D16 state that an area process can open, read and write any pseudo-terminal of the user under /dev/pts, host shells and other hubs' panes included, and the Goal's claim about secrets outside the area names this exception. Panes whose terminal tmux opens from the master descriptor (TIOCGPTPEER) with only /dev/tty granted are recorded under Open risks as the way to close it, to be probed on Neve before it is designed further.

Rationale: Option (a) of the hard-cap question, as the arbiter recommends: the record must not claim what Landlock cannot hold, and the mechanism is unmeasured.

### Added after Discussion (parent, 2026-11-26): A removed area file drops its registry entry

Decision: Every reader and writer of the area registry, lym area start and lym area stop included, drops a hub's registry entry whose area file is gone, so a hub whose area file is removed works as it does today with no extra verb.

Rationale: It holds the Goal's 'with the area off, everything works as it does today' without adding a host-side off verb, the narrower of the two options.

## Constraints

- Kernel facts, from `docs/research/platform/sandbox-without-new-kernel-2026-11-23.md` and its man-page sources:
  seccomp user notification is Linux 5.0, `pidfd_getfd` 5.6, `TSYNC_ESRCH` 5.7, `PIDFD_THREAD` 6.9;
  Landlock ABI 8 with `LANDLOCK_RESTRICT_SELF_TSYNC` measured on Neve's 7.0.0-31; OSL-1046 runs 7.0.0-42, ABI 8 unmeasured.
- Yama's `ptrace_scope` is 1 on both machines' Ubuntu default, so the supervisor reads an orphan's memory only as its subreaper (measured: `EPERM` without, ok with).
  A process that names a tracer with `PR_SET_PTRACER` admits it (kernel Yama documentation).
  Measured on Neve with a cgo-built probe whose parent starts a target and a tracer as siblings:
  the tracer's `pidfd_getfd` and open of `/proc/<target>/mem` fail with `EPERM` with no declaration,
  and succeed once the target calls `prctl(PR_SET_PTRACER, <tracer pid>)`, also after the target `exec`s `/bin/sleep`.
- lym is built with cgo (`tools/deploy` pins `CGO_ENABLED=1`), under which `syscall.AllThreadsSyscall` refuses; both restrictions use `TSYNC` for that reason.
  Measured on Neve with a probe built with `CGO_ENABLED=1`, holding six goroutines locked to their own threads:
  `prctl(PR_SET_NO_NEW_PRIVS)` on the calling thread sets `NoNewPrivs: 1` in that thread's `/proc/self/task/<tid>/status` alone;
  `landlock_restrict_self` with `LANDLOCK_RESTRICT_SELF_TSYNC` then succeeds, every thread's status shows `NoNewPrivs: 1`,
  and a write the ruleset handles fails with `EACCES` from each locked thread.
  With Landlock first, `landlock_restrict_self` fails with `EPERM` and no thread is restricted.
  `SECCOMP_FILTER_FLAG_TSYNC` puts the filter on every thread (seccomp(2)).
- tmux tries the session bus to put each pane in a scope; refused, it keeps the pane in the server's cgroup (measured).
- tmux's `detach-client -E` replaces a client with a shell command, and `-P` signals the client's parent (tmux man page, `detach-client`).
- `clone3` with `CLONE_INTO_CGROUP` places a child in the cgroup a descriptor opened `O_RDONLY` or `O_PATH` names (clone(2)),
  under the cgroup write check on `cgroup.procs`, which Landlock does not mediate, since it registers no `inode_permission` hook
  (`security/landlock/fs.c`, read and not measured).
  On Neve, `app.slice/cgroup.procs` under `user@<uid>.service` is owned by the user, and a pane's scope sits under `app.slice`.
  Outside tests, Go uses `clone3` only for `SysProcAttr.UseCgroupFD` and `CLONE_NEWTIME` (`syscall/exec_linux.go`),
  and glibc falls back to `clone` when `clone3` fails with `ENOSYS`.
- On Neve, `~/.claude/plugins/known_marketplaces.json` names the loomyard marketplace as a `directory` source at the loomyard hub's prime worktree,
  so D2 refuses that hub's area until the marketplace points at a clone outside it.
- The pane cap is made by the lym process that launches the pane, through `busctl`, with an uncapped fallback and a Warn;
  `RunMemoryCapped` uses `systemd-run --user --scope` with an `RLIMIT_DATA` fallback.
- Claude Code reads `CLAUDE_CONFIG_DIR` and `CLAUDE_CODE_TMPDIR`: both names are in the installed binary's strings on Neve.
  reed's server spawn scrubs both from its environment, so they reach a pane only through the launch line.
- Under the filter, `SCM_CREDENTIALS` names the connecting child, not the supervisor
  (the study's table row "peer as the test socket's server sees it"), and `serveHubCall` in `internal/creelengine/receive.go`
  serves the cross-hub `deliver` and `confirm` reading no peer identity.
- Name lookups on Neve: `/etc/nsswitch.conf` reads `hosts: files mdns4_minimal [NOTFOUND=return] dns`, with no `resolve` module,
  and `/etc/resolv.conf` links to `../run/systemd/resolve/stub-resolv.conf`, the stub at 127.0.0.53.
  So glibc resolves through `dns` to the stub over UDP, which the supervisor makes,
  and `mdns4_minimal`'s connect to avahi's socket is refused and falls through.
- `PATTERN-shell-guard-table`: the area adds no shellguard row.

## Auto-mode assumptions

- The entry's "config module `area`" becomes a file outside the hub (D2).
- Network egress stays open; stage 2 keeps only its token rule (D11).
- The system bus stays off the allow list (D5).
- Within a hub, creel's sender rules stay as today (D8).
- The landing deploy needs the operator's host-side command (D13).
- The AppArmor fallback is built only when the OSL-1046 probe fails (D14).

## Open risks

## Acceptance criteria

- D8: inside an area the outbound worker does not start another hub's daemon, and a hub without an area does not start an area hub's daemon.
- D8: an area hub whose prime's origin is rewritten to a repository the receiving hub trusts has its orch's mail stamped under the area file's `repository`,
  untrusted unless that repository is listed.
- Review: D14 — the note's split into tasks follows D14's order, with D8's creel identity ahead of the boot.
