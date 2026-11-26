---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:129
    summary: "D7 exempts host-side `lym reed attach`, which D6 and D13's `window` rely on, but attach runs `tmux -V` from the hub's agent-written `reed.tmux` key, reads hub state and spawns a host-side watchdog with the hub's `tmux` and `shell` keys before any ruleset applies, so the operator's attach runs agent-chosen binaries on the host."
    fix: "In D7 (and D6's attach bullet), state that an area hub's host-side attach reads no hub config or hub state before it restricts itself: it takes tmux from `PATH` or the area file's `tmux`, as D3's boot does, finds the socket from the area root, skips `EnsureSession`, `Status` and the watchdog spawn (the watchdog runs inside the area or not at all), and applies the ruleset and filter before it runs any tmux. Add an acceptance criterion: with a hub `reed.tmux` naming a script under the hub, a host-side attach of an area hub never runs it."
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:110
    summary: "D6 puts the supervisor's socket, the hub's tmux socket and its creel socket under the area root, where agents hold every right; an agent can unlink the supervisor socket and bind its own, so a host-side attach hands its filter listener to the agent, which answers CONTINUE and lets a `detach-client -E` command reach the bus, and a replaced creel socket answers other hubs' `confirm` and forges cross-hub mail as this hub's orch."
    fix: "In D3 and D6, move the supervisor's socket out of the area root into the area state directory (read-only inside; connect needs no Landlock write right on ABI 8), and state that every host-side caller of a supervisor verb (`lym area start`, attach, `lym area stop`) checks the peer's SO_PEERCRED pid against the supervisor record's pid before it sends anything. In D8, state how a daemon that calls `confirm` on an area hub's socket knows the answer comes from that hub's daemon (for example, `lym area start` records the daemon's expected peer, or the confirm caller checks SO_PEERCRED against the inside test of that hub). Add acceptance criteria for a replaced supervisor socket and a replaced creel socket."
  - id: F3
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:123
    summary: "D6's attach client is the operator shell's child, not the supervisor's descendant, so under Yama `ptrace_scope` 1, which the record's own Constraint pins (EPERM without ancestry), the supervisor can neither read the client's `/proc/<tid>/mem` nor take its socket with `pidfd_getfd`; every connect of the client, its own tmux socket included, fails and the attach cannot work."
    fix: "In D6, have the attach process name the supervisor as its tracer with `prctl(PR_SET_PTRACER, <supervisor pid>)` before it installs the filter (or have the supervisor spawn the client), state the Yama rule that makes this work with its source (kernel Yama documentation), and measure it with the probe on Neve; record the result in D6 or the Constraints."
  - id: F4
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:115
    summary: "D6 grants every right on the Go build and module caches, by default the host's `~/.cache/go-build` and `~/go/pkg/mod`, which host-side builds and D13's `lym area deploy` with the host's toolchain read; an agent plants a cache or module entry that the host compiles into an installed binary, persistence against the Goal, and D2's fence does not list the caches."
    fix: "In D6, give the area its own `GOCACHE` and `GOMODCACHE` under the hub (set on the server's environment like `TMPDIR`), deny write on the host's caches, and add the host's caches to D2's list of host-side inputs the start check refuses; state in D13 that `lym area deploy` builds with the host's own caches, never the area's. Extend the D6 acceptance criterion to a write into the host's Go caches failing inside."
  - id: F5
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:131
    summary: "D7 refuses only a verb whose cwd resolves to the area hub, but host-side verbs reach area hubs through the machine record: `lym machine start` pulls each hub's board and opens its VS Code workspace and attach windows, the machine sweeps walk every recorded hub, and `lym hibernate request` pauses every session, so host lym runs git over area repositories whose `.git/config` and hooks agents write."
    fix: "In D7, extend the refusal from the cwd's hub to every hub a host-side verb acts on: any lym code path that resolves an area hub from the machine record or a shortname, outside the area, skips that hub with a notice naming the way forward, and `lym machine start` skips area hubs. Name the routes by group (machine-record walks, cross-hub creel geometry, hibernate's session pause) and add an acceptance criterion that `lym machine start` from another hub runs no git over an area hub."
  - id: F6
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:116
    summary: "D6 places the area root at `/tmp/lyma-<uid>-<hash>`, a temp directory outside `internal/lymtemp` and named outside `lymdirs`, and points `TMPDIR` at it, while PATTERN-lym-files binds temp under `lymtemp` and says no code hands a child a `TMPDIR` that is not a run directory; the record declares no change to that entry and says nobody creates the root or checks its owner, mode or link status in world-writable `/tmp`."
    fix: "In D6, have `internal/lymtemp` own the area root (a per-hub root beside `lym-<uid>`, created 0700 by the supervisor and refused unless it is a real directory owned by the user, as the lym root is), and point `TMPDIR` at a lymtemp run directory under it; or declare a `PATTERN-lym-files` change in `## PATTERN changes` whose text admits the area root and its `TMPDIR`. State who creates the root and what the start refuses."
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:65
    summary: "D3's supervisor spawns the server from the operator's host-shell environment, skipping reed's scrub, `stripTraceID` and `stripIdentityEnv`, and no step sets `TMPDIR`, `GOTMPDIR` and `TMUX_TMPDIR` or the `-S` socket argv; every pane then inherits `DBUS_SESSION_BUS_ADDRESS`, `SSH_AUTH_SOCK` and any `GH_TOKEN`, which `githubclient` tries first, against D11."
    fix: "In D3's `boot` bullet, state that the supervisor builds the server's environment: reed's scrub and identity strips applied, `GH_TOKEN`, `GITHUB_TOKEN` and the session's bus and agent addresses removed, `TMPDIR`, `GOTMPDIR` and `TMUX_TMPDIR` set per D6, and the socket path in the argv equal to what host-side attach derives. Add an acceptance criterion that a pane's environment holds none of the removed names."
  - id: F8
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:67
    summary: "D3 orders `lym area exec` as Landlock with TSYNC, then `no_new_privs`, then the filter, but landlock_restrict_self(2) refuses with EPERM unless the calling thread already has `no_new_privs`; and the Constraints leave whether TSYNC carries `no_new_privs` to every thread of lym's cgo build unmeasured, a fact the boot rests on deferred to the implementer."
    fix: "In D3, order the steps as `prctl(PR_SET_NO_NEW_PRIVS)` on the calling thread, then Landlock with `LANDLOCK_RESTRICT_SELF_TSYNC`, then the filter with `SECCOMP_FILTER_FLAG_TSYNC`, citing landlock_restrict_self(2). Replace the Constraints line 'unmeasured in lym's cgo build' with the measured result: run the probe's restriction path from a cgo-built binary on Neve and record whether each thread's `/proc/<pid>/task/<tid>/status` shows `NoNewPrivs: 1` and both restrictions, with the command and its output."
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:145
    summary: "D8's refusal of every verb but `deliver` and `confirm` from another area fails open: a sender that exits before the daemon checks leaves a dead `SCM_CREDENTIALS` pid whose ancestry reaches no supervisor, and the operator's attach client, made to run a command by `detach-client -E`, carries a filter but no supervisor ancestry; both resolve to `EXTERNAL`, which D8 calls an act outside the area."
    fix: "In D8, state that a daemon refuses every verb but `deliver` and `confirm` from a peer whose pid is gone, unreadable or carries any seccomp filter it cannot place outside every area, rather than only from a peer proven inside; name the pid read (for example `SCM_PIDFD`, which pins the process) and add an acceptance criterion for a sender that exits before the daemon reads it."
  - id: F10
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:145
    summary: "D8 has every daemon apply D3's inside test to a peer from another hub's area, which needs that hub's supervisor record, but D6 grants an in-area daemon read on its own supervisor record only, and no decision says how a daemon finds every area hub's supervisor."
    fix: "In D8 (and D6's read list), state where a daemon reads the supervisor records of every area hub, such as the area state directory's parent granted read inside every area, or a supervisor pid carried in each area hub's machine-record entry written by `lym area start`."
  - id: F11
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:100
    summary: "D5 admits another hub's creel socket by the path its machine-record entry carries, read at each decision, but any process outside an area writes that shared file; an entry naming `/run/user/<uid>/bus` with the area-aware level puts the bus on the allow list, against D5's 'never a bus', and no check bounds the path's shape."
    fix: "In D5, state that a derived creel entry is admitted only when its canonical path has the creel socket name of that entry's hub (`creel-<shortname>-<h>.sock` with `<h>` from its hub path) and sits in a lym temp root or an area root, and add an acceptance criterion with a record entry naming the bus."
  - id: F12
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:90
    summary: "D4 lets every TCP and UDP connect through, loopback included, so any host service on 127.0.0.1 or ::1 (a debug port, a dev server, an editor bridge) is reachable unmediated, while the Goal claims agents cannot act outside the hub and D16's non-claims name only the open network and datagram sends."
    fix: "Either have D4's supervisor refuse loopback and link-local connects off an area-file list, with an acceptance criterion, or add loopback services to D16's non-claims and narrow the Goal's sentence to match."
  - id: F13
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:154
    summary: "D9 routes only scope creation through the supervisor, but after a SIGKILL or an OOM kill `RunMemoryCapped` ends the scope's leftovers by writing `cgroup.kill` under `/sys/fs/cgroup`, which D6's read-only `/sys` refuses, and the held-child flow produces no marker naming the scope's cgroup that today comes from `scopeStartScript`."
    fix: "In D9, state how the run kind reports its scope's cgroup to the caller and who ends the scope's tree after a kill: the `cap-memory` verb answers the cgroup path and a further supervisor verb kills a scope of the caller's tree, or D6 grants write on the area's own scopes' `cgroup.kill`. Add the tree kill to D9's acceptance criterion."
  - id: F14
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:161
    summary: "D9's bound checks only that the pid descends from the supervisor, which also admits the supervisor's own host-side children (D12's relay, D13's Konsole windows); an area agent can cap them with a tiny limit and kill or starve host processes."
    fix: "In D9's bound, require the pid to pass D3's inside test (carries the hub's barrier and descends from the supervisor), not descent alone, and extend the acceptance criterion to a refused cap on the relay's pid."
  - id: F15
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:166
    summary: "D10 sets `CODEX_HOME` under the hub but is silent that codexengine's home preparation links the lym home's `auth.json` and `sessions` into the operator's `~/.codex` and takes the home from `codex.yaml`'s `home` key; inside the area those links dangle, or if granted they write the operator's home, so codex lacks the coverage D10's parity rationale claims."
    fix: "In D10, state that inside an area codexengine ignores `codex.yaml`'s `home`, makes no links into the operator's Codex home, and keeps `auth.json` and `sessions` in the area's own directory; add a codex row to D10's acceptance criterion."
  - id: F16
    severity: MEDIUM
    class: consistency
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:190
    summary: "D12 says the supervisor-started relay holds no `runproc` record, but the relay registers itself under `runproc.HubOwner` with role `slack-relay` at start, `StopRelay` ends it through that record, and PATTERN-child-processes' background says the relay sits under `HubOwner`."
    fix: "In D12, state what happens to the relay's self-registration under an area: the relay started by the supervisor registers no record (and the background line changes) or keeps it and `slack-relay stop` ends it through the supervisor; drop the sentence that contradicts the code."
  - id: F17
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:192
    summary: "D12's bound says the relay reads only the slack module and its own state, but that state (the setup file's operator id, channel and bridge maps, outbox, forwarding file) sits in agent-writable hub paths, so a host process holding the operator's Slack tokens acts on area-written input, such as a different operator id or queued posts."
    fix: "In D12's bound, state which of the relay's inputs are agent-writable under an area and what bounds each: move the operator id and channel map out of the hub (for example into the area state directory), or state the posts and recipients an area-written outbox can reach as a non-claim in D16."
  - id: F18
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:206
    summary: "D13's bound says no verb runs area-chosen code on the host, but `lym area deploy` runs the deploy command in a clone of a commit agents pushed, and lym's deploy is `tools/deploy` from that repository, so area-written code runs on the host during the deploy, not only once installed as D16's non-claim says."
    fix: "Restate D13's bound and D16's non-claim to say that `lym area deploy` runs the landed commit's own code on the host when the operator runs it, or narrow the deploy so it runs no repository code (for example `go build` of a fixed package with the host's caches and no `go run`)."
  - id: F19
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:94
    summary: "D4 sends a hub whose supervisor died to `lym area stop` then `lym area start`, but D3's stop ends 'every process left in the supervisor's tree', a tree that is gone once its members are reparented to `systemd --user`; dobby and the creel daemon live on with every connect failing ENOSYS while holding their locks, and the next start boots beside them."
    fix: "In D3, state how `lym area stop` finds the processes of a dead supervisor's area (for example every process carrying the hub's barrier whose recorded cgroup or start sits under the record) and ends them, and add an acceptance criterion: after the supervisor is killed, stop then start leaves one dobby and one creel daemon, both answering."
  - id: F20
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:73
    summary: "D3's host-side `lym area stop` ends the hub's runs through `runproc` records stored in the hub's `.lym`, which agents write, so an agent can list any host pid there and have the host-side stop signal it, against D7's rationale that host lym never acts on agent-written hub state."
    fix: "In D3, state that `lym area stop` signals only processes that pass the hub's inside test or descend from the recorded supervisor, whatever the run records name, and add that to the D3 stop criterion."
  - id: F21
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:52
    summary: "D2's start check covers only the area file's `write` list, but D6 also grants every right on the derived hub root, so a hub root at or above `$HOME`, or one holding a host-side input, passes the check and breaks D2's bound."
    fix: "In D2, extend the start check to every write grant, derived ones included: the hub root and the area's caches are refused when on, under or above a host-side input, and the D2/D5 acceptance criterion covers a hub root above `~/.config`."
  - id: F22
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:51
    summary: "D2 has `lym area init` write the Landlock study's read list, which grants every right on `/tmp` and the host's Go caches and read on `~/.claude` and `~/.claude.json`, against D6's 'no write on `/tmp`' and D10's own config directories."
    fix: "In D2, state that `lym area init` writes the read, exec and write lists D6 defines (system directories, toolchain, launcher, `lym`, git config, machine record) and none of the study's `/tmp`, Go-cache or `~/.claude` rows."
  - id: F23
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:223
    summary: "D15 renders the AppArmor profile from 'the same allow list', named by a hash of its text and loaded once by root, while D5 derives other hubs' creel sockets from the machine record at each decision; either every hub that records itself changes the hash and the start refuses until IT reloads, or cross-hub creel freezes at load, and neither is stated."
    fix: "In D15, state which derived entries the profile renders (for example a path pattern over every hub's creel socket name, so the text does not change when a hub records itself) and how a new hub reaches an AppArmor area."
  - id: F24
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:274
    summary: "Open risks call sender confirmation through a supervisor-made connect unmeasured, but `deliver` and `confirm` are served before any sender resolution (`serveHubCall`) and the probe measured `SCM_CREDENTIALS` naming the child, so the line restates a settled fact as a risk."
    fix: "Drop the second Open risks line, or replace it with the measured result: SCM_CREDENTIALS names the connecting child under the filter (study table row 'peer as the test socket's server sees it'), and cross-hub `deliver` and `confirm` read no peer identity."
  - id: F25
    severity: LOW
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:275
    summary: "Open risks leave name lookups unmeasured, a fact the area's builds and HTTPS rest on; Neve's `/etc/nsswitch.conf` reads `hosts: files mdns4_minimal [NOTFOUND=return] dns`, with no `resolve` module, and `/etc/resolv.conf` links to the stub at 127.0.0.53, so glibc never asks systemd-resolved's socket and the line's premise is wrong."
    fix: "Replace the third Open risks line with a Constraint: on Neve glibc resolves through `dns` to the 127.0.0.53 stub over UDP, which the supervisor makes, and `mdns4_minimal`'s avahi socket is refused and falls through (source: Neve's `/etc/nsswitch.conf` and `/etc/resolv.conf`); state the same read for OSL-1046 as an item of D14's step 1 probe run."
  - id: F26
    severity: LOW
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:172
    summary: "D10 says the host's plugin cache is granted read-only and linked into the area's Claude directory, and D11 says an empty helper value clears the host's credential helpers, both resting on tool behaviour stated without a pin: whether Claude Code writes into its plugin directory at start or update, and git's rule for an empty `credential.helper`."
    fix: "In D10, record whether Claude Code writes under the plugin directory (observed on Neve, with the command and result) and what fails if it does; in D11, cite gitcredentials(7) or git-config(1) for the empty `credential.helper` reset."
---

### [BLOCKING] Host-side attach runs hub-config binaries before the area applies

**Location:** `_lym/discussion/decision-record.md:129` (D7), with D6's attach bullet and D13's `window`.
**Issue:** `internal/reedcli/attach.go` runs `CheckTmuxFloor(c.eng.TmuxPath())`, `EnsureSession`, `Status`, `ensureWatchdogSpawned` and `handOver(c.eng.TmuxPath(), …)` before anything restricts the process. The tmux path and the watchdog's shell come from the hub's `reed` config, which agents write. D3's "never from hub config" covers only the supervisor's argv. Three lenses traced the same path.

### [BLOCKING] Agents can replace the supervisor's and the creel daemon's sockets under the area root

**Location:** `_lym/discussion/decision-record.md:110` (D6), with D3 and D8.
**Issue:** D6 puts the supervisor's socket under the area root, where agents hold every right, and nothing on the host authenticates the peer. An impostor receives the attach client's filter listener, and the client's connects then go wherever the impostor says. The same applies to the hub's creel socket, whose path `lym area start` writes into the machine record. `Deliver` in `internal/creelengine/receive.go` asks the sending hub's socket to `confirm`. An agent answering there confirms forged mail from `<hub>:orch`, and a hub that trusts this one stamps that mail trusted orch.

### [BLOCKING] Yama stops the supervisor serving the host-side attach client

**Location:** `_lym/discussion/decision-record.md:123` (D6).
**Issue:** The record's own Yama Constraint and the probe's orphan row show that the supervisor reads memory and takes descriptors only from its descendants. A host-shell attach is not one of them.

### [BLOCKING] The area's Go caches are the host's

**Location:** `_lym/discussion/decision-record.md:115` (D6), with D2 and D13.
**Issue:** D6 grants every right on the Go caches. Host builds and `lym area deploy` read them, so cache content an agent plants ends up in host binaries.

### [BLOCKING] D7's refusal misses host verbs that reach area hubs through the machine record

**Location:** `_lym/discussion/decision-record.md:131` (D7).
**Issue:** `lym machine start` (`internal/machinecli/cli.go`) works from any directory. It pulls each recorded hub's board, starts its worktrees and opens its VS Code workspace. `hubsToSweep` walks every recorded hub. None of these routes passes through D7's check on the cwd's hub, so host git runs over repositories that agents write.

### [BLOCKING] The area root sits outside `lymtemp`, and nothing creates it or checks its owner

**Location:** `_lym/discussion/decision-record.md:116` (D6).
**Issue:** `PATTERN-lym-files` (overview: temp under `lymtemp`; background: no code hands a child a `TMPDIR` that is not a run directory) binds here, and the record declares no change to it. `internal/lymtemp/doc.go` shows the owner and mode checks the area root lacks in world-writable `/tmp`. The walkthrough lens raised the `TMPDIR` half; the handler adds the placement and the creation.

### [MEDIUM] The supervisor's server spawn has no environment hygiene

**Location:** `_lym/discussion/decision-record.md:65` (D3).
**Issue:** `internal/reedengine/lifecycle.go:461-477` scrubs the server's environment today. D3's supervisor path names no such step, and no step sets the temp variables D6 relies on.

### [MEDIUM] D3's restriction order contradicts landlock_restrict_self(2), and the cgo TSYNC fact is deferred

**Location:** `_lym/discussion/decision-record.md:67` (D3), with the Constraints' cgo line.
**Issue:** landlock_restrict_self(2) refuses with EPERM unless the calling thread already has `no_new_privs`. The probe sets it first, with an all-threads call (`seccomp-connect-probe/main.go:845-848`). The record orders it after Landlock and leaves the cgo-build behaviour unmeasured, so the rubric's deferred-research rule applies.

### [MEDIUM] D8's cross-area refusal fails open

**Location:** `_lym/discussion/decision-record.md:145` (D8).
**Issue:** `resolveSender` in `internal/creelengine/identity.go` reads any process in no pane as `EXTERNAL`. A dead pid, or the attach client's command, falls outside the inside test and reaches that path.

### [MEDIUM] No daemon can read other areas' supervisor records

**Location:** `_lym/discussion/decision-record.md:145` (D8).
**Issue:** D6 grants an in-area daemon read on its own supervisor record only, but D8's inside test needs every area's record.

### [MEDIUM] Derived creel paths are taken from a file any process outside an area writes

**Location:** `_lym/discussion/decision-record.md:100` (D5).
**Issue:** `internal/hubregistry/hubregistry.go` writes `hubs.json` from every daemon, under no area. D5 states no check that a derived path is a creel socket.

### [MEDIUM] Loopback services are reachable and unclaimed

**Location:** `_lym/discussion/decision-record.md:90` (D4).
**Issue:** The supervisor makes every TCP and UDP connect, loopback ones included. The Goal's claim and D16's non-claims both leave them out.

### [MEDIUM] The run scope's kill and its cgroup report are missing under D9

**Location:** `_lym/discussion/decision-record.md:154` (D9).
**Issue:** `killScope` and `scopeCgroup` in `internal/proc/memorycap_linux.go` need a `cgroup.kill` write and the start marker, and D9 supplies neither inside the area. Every lens flagged it; bound-breaker rated it LOW, but a capped gate whose children outlive the cap is a regression from today, so it is kept at MEDIUM.

### [MEDIUM] `cap-memory` admits the supervisor's host-side children

**Location:** `_lym/discussion/decision-record.md:161` (D9).
**Issue:** The descent check alone admits the relay and the Konsole windows, which run outside the area.

### [MEDIUM] Codex's home links reach the operator's Codex home

**Location:** `_lym/discussion/decision-record.md:166` (D10).
**Issue:** Per `internal/shuttleengine/codexengine/doc.go`, the lym home links `auth.json` and `sessions` to the operator home's. D10 is silent about those links.

### [MEDIUM] D12's runproc claim contradicts the relay's registration

**Location:** `_lym/discussion/decision-record.md:190` (D12).
**Issue:** The relay registers under `runproc.HubOwner`, as `PATTERN-child-processes`' background records.

### [MEDIUM] The relay acts with host secrets on area-written state

**Location:** `_lym/discussion/decision-record.md:192` (D12).
**Issue:** The relay's setup, maps and outbox sit in the hub, which agents write.

### [MEDIUM] `lym area deploy` runs area-written code on the host

**Location:** `_lym/discussion/decision-record.md:206` (D13).
**Issue:** The deploy command runs the pushed repository's own tooling, so D13's bound overstates.

### [MEDIUM] A dead supervisor's tree has no defined stop

**Location:** `_lym/discussion/decision-record.md:94` (D4), with D3's stop.
**Issue:** Once the supervisor dies its tree is reparented, so stop-then-start leaves the old daemons running with every connect failing.

### [MEDIUM] `lym area stop` trusts agent-written run records

**Location:** `_lym/discussion/decision-record.md:73` (D3).
**Issue:** A host-side stop signals whatever pids the hub's `runproc` records name, and agents write those records.

### [MEDIUM] D2's check ignores the derived write grants

**Location:** `_lym/discussion/decision-record.md:52` (D2).
**Issue:** The check covers only the area file's write list, while D6 also grants every right on the derived hub root.

### [MEDIUM] `lym area init`'s content contradicts D6

**Location:** `_lym/discussion/decision-record.md:51` (D2).
**Issue:** The study's list grants `/tmp`, the Go caches and `~/.claude`, all of which D6 and D10 take away.

### [MEDIUM] D15's profile hash against D5's per-decision derived list

**Location:** `_lym/discussion/decision-record.md:223` (D15).
**Issue:** A profile loaded once by root cannot follow a list that changes whenever a hub records itself.

### [LOW] An Open risk restates a settled fact

**Location:** `_lym/discussion/decision-record.md:274` (Open risks).
**Issue:** `serveHubCall` in `internal/creelengine/receive.go` serves `deliver` and `confirm` with no sender resolution, and the probe's table already shows `SCM_CREDENTIALS` naming the child.

### [LOW] The resolver risk is a deferred, answerable fact with a wrong premise

**Location:** `_lym/discussion/decision-record.md:275` (Open risks).
**Issue:** Neve's `hosts:` line has no `resolve` module, and its `resolv.conf` points at the 127.0.0.53 stub.

### [LOW] Unpinned tool facts in D10 and D11

**Location:** `_lym/discussion/decision-record.md:172` (D10), with D11.
**Issue:** Neither the plugin cache's read-only behaviour nor git's empty-helper reset cites an observed result or a document.

## Rejected

- bound-breaker F1 (a nested filter's listener answering CONTINUE bypasses the supervisor): seccomp(2) states that at most one filter using `SECCOMP_FILTER_FLAG_NEW_LISTENER` can be installed for a thread, so an agent under the area's filter cannot add a second listener. An extra filter returning a higher-precedence action refuses the call and never lets it run.
- interactions F9 (an older daemon's `hubregistry.Record` strips other entries' new fields): the existing-paths lens traced `state.UpdateJSON` keeping unknown keys on entries it does not update, as `PATTERN-tolerant-records` binds.
- interactions F13's `lym hibernate request` half: D13 routes hibernate through a verb, and pausing sessions is creel traffic, not host-side reading of hub config. Its `lym machine start` half is kept in F5.
- The OSL-1046 probe under Cortex XDR, raised by the focus as a possible deferred research question: it is an operator action that D14's step 1 branches on, so it is no finding.
- D15's unmeasured AppArmor premises, raised by the same focus entry: they rest on the study's reading of AppArmor's documentation, and measuring them needs a root profile load the writer could not make.
