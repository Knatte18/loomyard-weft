## Goal

Stage 2 of the closed area: window, hibernate and deploy as supervisor verbs, and lym ide refused inside

Takes: the core part's supervisor verb protocol and attach client; termwindow's Launcher.OpenAttach, hibernatecli's request and its systemctl spawn, battenshed's LandDeps.RunDeploy and its wiring, idecli; the operator's stop of D14 step 3
Delivers: the supervisor's window verb (direct child of the hub root, title built from the directory name, Konsole running the host's lym reed attach with no shell after it) with lym reed window routed to it inside an area; the hibernate verb running systemctl hibernate alone; the deploy verb recording a pending deploy from Batten's Land row with a notice; lym area deploy cloning https://github.com/<repository>.git into the area state directory, fetching the area file's deploy_branch, checking the commit is on it and running the area file's deploy command with the host's toolchain and caches, refusing with the way forward when repository, deploy or deploy_branch is missing, and lym area init writing deploy_branch as main (Added after Discussion, with its acceptance criterion that a rewritten origin and a hub record naming another branch change neither); lym ide refusing inside an area
Exports: the window, hibernate and deploy verbs and their clients

## Scope

The part changes these packages:

- internal/areaengine (about 300 changed lines)
- internal/areacli (about 200 changed lines)
- internal/termwindow (about 50 changed lines)
- internal/reedcli (about 40 changed lines)
- internal/hibernatecli (about 40 changed lines)
- internal/battenshed (about 60 changed lines)
- internal/battencli (about 40 changed lines)
- internal/idecli (about 20 changed lines)

## Decisions

### D13 — Host exits as fixed supervisor verbs

The supervisor serves, besides D3, D6, D9 and D12, verbs that take no command line from the area:

- `window`: a worktree whose canonical parent is the hub root, never a deeper path, and no title:
  the supervisor builds the title from the worktree's directory name, keeping only letters, digits, dot, underscore and hyphen.
  It runs Konsole with the host's `lym reed attach`, which runs its client under the area (D6), and no shell after it.
  `lym reed window` in `internal/termwindow/termwindow.go` routes to it inside an area; no hub config key, the post-window command included, applies to it.
- `hibernate`: `lym hibernate request` pauses every session as today and then asks the verb, which runs `systemctl hibernate` and nothing else.
- `deploy`: Batten's Land row (`internal/battenshed/land.go`) records a pending deploy of the landed commit through the verb and goes on, with a notice naming `lym area deploy`.
  The operator runs `lym area deploy` from a host shell; it shows the pending commit, fetches the parent branch from origin into its own clone in the area state directory,
  checks the commit is on that branch, and runs the area file's deploy command there with the host's toolchain and the host's own Go caches, never the area's.
- `lym ide` refuses inside an area, naming the reason: VS Code on the host runs git over repositories the area writes.
Bound: the only area-written code a verb runs on the host is the landed commit's own, its deploy tooling included,
which `lym area deploy` runs when the operator runs it (D16); nothing deploys without the operator's command.
Any area process can hibernate the host or open an attach window; neither runs area code outside the area.
Nothing from the area reaches Konsole's arguments but the canonical worktree path and the title the supervisor builds from its name.
Rationale: these are the bwrap design's stage 3 exits, still broken inside this area, and a deploy on request would run any commit an agent pushed.

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

- D3, D7, D13: with `<hub>/x/wt` holding a `.git` file and `<hub>/x` holding a `_bolt/_lym` and a reed config whose tmux names a script,
  `window` and `boot` refuse `<hub>/x/wt`, and a host lym verb run in it refuses with the way forward, never running the script or any git.
- D13: a `window` request for a worktree whose name holds `;` or `=` opens a window whose Konsole arguments carry neither character in the title.
- D13: the `window` verb refuses a worktree that is no direct child of the hub root; the Land row's deploy inside an area records a pending deploy and runs nothing on the host; `lym area deploy` refuses a commit not on origin's parent branch; `lym ide` inside an area refuses with its reason.
- D13: `lym hibernate request` inside an area reaches `systemctl hibernate` only through the `hibernate` verb.
