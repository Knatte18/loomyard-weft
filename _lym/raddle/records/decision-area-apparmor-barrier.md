## Goal

The closed area's AppArmor fallback barrier, built only when the OSL-1046 probe fails

Takes: the core part's supervisor, launcher, attach client, area file with its barrier key, and the start check's barrier hook; the OSL-1046 probe result of D14 step 1
Delivers: lym area profile rendering the hub's profile from the area file's lists and the Landlock paths, with other hubs' creel sockets as one creel-*.sock pattern and the name a hash of the text, and naming the root load command; the launcher and attach client entering it through aa-exec, each still installing a filter with D4's foreign-ABI, io_uring_setup and clone3 rules, no connect rule, no listener, and TSYNC with TSYNC_ESRCH (Added after Discussion); the start refusing while no loaded profile carries the rendered name; the inside test's apparmor label branch exercised end to end
Exports: areaengine's apparmor barrier and its profile renderer

## Scope

The part changes these packages:

- internal/areaengine (about 350 changed lines)
- internal/areacli (about 100 changed lines)

## Decisions

### D15 — The AppArmor profile per hub as the fallback barrier

With `barrier: apparmor` in the area file, `lym area profile` renders a profile for the hub from the area file's lists and the Landlock paths,
denying every Unix-socket connect off the allow list, named with a hash of its rendered text, and names the root command that loads it.
Other hubs' creel sockets enter it as one path pattern, `creel-*.sock` directly in a lym temp root or an area root,
so its text does not change when a hub records itself, and a new hub is reachable from an AppArmor area with no reload.
The text holds no other hub's entry, since no area is granted read on another hub's files (D6), so a new area hub changes no other hub's profile hash.
The supervisor starts its launcher (D3) with `aa-exec -p <profile>`, and the attach client of D6 enters the profile too;
both still install a filter holding D4's foreign-ABI, `io_uring_setup` and `clone3` rules, with no `connect` rule and no listener;
Landlock still applies, and the supervisor still serves D3's `boot`, D6's `attach-scope`, D9, D12 and D13.
The start refuses when no loaded profile in `/sys/kernel/security/apparmor/profiles` carries the name the current file and geometry render.
This barrier meets each clause of `PATTERN-closed-area`: the hub boots only via its supervisor, as on the seccomp route;
the profile is its one connect barrier beside its one ruleset;
the profile decides every connect, from the paths `lym area profile` renders and the area file only `areaengine` reads;
and every host exit stays a supervisor verb.
Bound: unmeasured on both machines; each new area hub's own profile and each changed list is one more root load, by IT on OSL-1046.
The pattern admits every hub's creel socket, so D5's area-aware gate does not apply under this barrier, and a daemon of an older binary is reachable.
Rationale: the study's ranking keeps it for the case the supervisor fails on OSL-1046 and IT will load profiles.

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

- D3, D4, D15: under each barrier, a pane process that calls `clone3` with `CLONE_INTO_CGROUP` into a cgroup outside the area's scopes fails,
  and no process of it leaves those scopes.
- D3, D15: the inside test holds for a pane process under each barrier, and fails for a host process and for the supervisor's host-side children.
- D15: with `barrier: apparmor`, `lym area profile` renders a profile denying the session bus and other hubs' tmux sockets, whose text is unchanged when another hub records itself or another area hub starts, and `lym area start` refuses while no loaded profile carries its rendered name.
