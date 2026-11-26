---
verdict: BLOCKING
findings:
  - id: F33
    severity: BLOCKING
    class: design
    carries: F27
    location: _lym/discussion/decision-record.md:96
    summary: "D3's inside test still fails open. It rests on no area process leaving its scope because /sys is read-only, but clone3 with CLONE_INTO_CGROUP places a child in another cgroup through an O_RDONLY or O_PATH directory descriptor (clone(2)). Landlock registers no inode_permission hook, so it does not mediate the cgroup.procs write check, and the user owns cgroup.procs under user@<uid>.service. D3's filter allows clone3. A child keeps the filter and the Landlock domain, leaves every lym-area scope, passes D8 as a host process and survives lym area stop."
    fix: "In D4, have the filter refuse clone3 with ENOSYS, so callers fall back to clone, which names no cgroup (seccomp cannot read the clone3 argument struct, so the whole call is refused). State that D15's apparmor barrier installs that same filter rule, since it installs no connect filter. In D3, replace the /sys premise with the two bounds: writing cgroup.procs is denied by Landlock, and clone3 is refused by the filter. Record the evidence in Constraints. Add an acceptance criterion: under each barrier, a pane process that tries clone3 with CLONE_INTO_CGROUP into a cgroup outside the area's scopes fails, and no process of it leaves those scopes."
  - id: F34
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:193
    summary: "D7 says every host lym verb refuses over an area hub, but its one mechanism walks the cwd. lym reed window <hub>/<worktree> run from HOME passes it, loads the hub's desktop config through placeTerminalWindow and runs window_command, a key agents write, on the host (reedcli/window.go, desktophook/window.go). Verbs that take --hub-path have the same shape. After a refusal, the exit hook still resolves the cwd by its .git root, creates a trace file through whatever .lym/logs links to, and loads the hub's logger config (cmd/lym/exitsweep.go, internal/logger/sink.go NotifyExit)."
    fix: "In D7, state the check at the point where a hub is chosen, not only on the cwd: every hub path lym derives on the host, from the cwd, a path argument, a flag, the machine record or a shortname, passes the area-file check before any file under it is read or written. State that a refusal ends the process with no later step that resolves the cwd, reads hub config or writes under the hub, the exit log sweep included. Add acceptance criteria: with the area hub's desktop.machine.yaml naming a script, lym reed window on its worktree run from outside the hub refuses and never runs it; a refused verb run in an area hub creates no file under the hub."
  - id: F35
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:150
    summary: "D6's read grants cannot carry D8's cross-hub mail. A Landlock rule on one file follows its inode, and the machine record and area registry are replaced by rename on every write, so the grant is lost at the first rewrite. state.ReadJSON also opens hubs.lock, which is not granted. An in-area daemon further needs a non-area hub's Bolt shortname file (hubregistry.go:82) and prime .git/config (repository.go:38), which D15 says no area may read. The acceptance criterion for a post in both directions cannot pass."
    fix: "In D8, have an area daemon take every other hub's path, shortname, repository and socket path from host-written records alone, never from the other hub's files; the machine-record entry or the area registry must carry them for non-area hubs too, with the writer of each named. In D6, grant read on a directory that holds only the files an area may read (the machine record, its lock, the area registry), not on single files, and say the same for the token file, machine.json and the git config, or state how a replaced file is re-granted. Keep D15's rule that no grant lies on another hub's files. Add an acceptance criterion: a pane reads the machine record after a host daemon rewrote it."
  - id: F36
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:76
    summary: "D3 boots a worktree's session and watchdog from a child of the supervisor restricted as lym area exec restricts. That is a second landlock_restrict_self, so the watchdog and everything it spawns sit in a sibling domain of the tmux server's. Signal scope then refuses signals both ways, which D3's own Rationale and D1's one domain per hub rule out. A server booted again while dobby or creel survive gets another sibling domain too."
    fix: "In D3, give each hub one restriction that lasts as long as the supervisor: the supervisor starts one restricted launcher process, which applies Landlock and the filter once and then starts the server on every boot and each session and watchdog as its own children, so every process of the area descends from that one domain. State that boot asks the launcher and never restricts a new child. Add an acceptance criterion: a pane process can signal a watchdog started by boot, and still can signal a surviving dobby after the server is booted again."
  - id: F37
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:177
    summary: "D6 puts the operator's attach client under the area but keeps its host environment and terminal. lym reed attach execs tmux with os.Environ (reedcli/attach_unix.go), so a detach-client -E command an agent sends reads GH_TOKEN, SSH_AUTH_SOCK and every other exported secret. tmux's update-environment also copies client variables into the session at attach. The command's orphans keep the terminal descriptor after the attach ends, while the Konsole line then execs a host bash on that terminal (termwindow.go:161), so area code can read what the operator types there."
    fix: "In D6, state that lym reed attach gives its client an environment built from an allow list of terminal variables (TERM, locale and the like), never the host shell's, and that the area's tmux config empties update-environment. State that the supervisor watches the client's pid and ends the attach scope when it exits, before the terminal returns to a host process, and that a window opened for an area hub starts no host shell in the worktree. Add acceptance criteria: a detach-client -E command sees no GH_TOKEN or SSH_AUTH_SOCK set in the attaching shell; after it exits, no process of the attach scope holds the terminal."
  - id: F38
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:57
    summary: "D2's start check omits the operator's local plugin marketplaces, which host Claude sessions load plugins and hooks from. On Neve the loomyard marketplace is a directory source at the loomyard hub's prime worktree (~/.claude/plugins/known_marketplaces.json), so turning that hub's area on grants every agent write to code the host sessions then run: the persistence exit the Goal closes."
    fix: "In D2, add to the host-side inputs every local-directory marketplace path and install location the operator's engine configs name, read by lym area init and lym area start, so a hub that holds one is refused with its way forward (point the marketplace at a clone outside every area hub). Say in D10 that the read-list entry is for a marketplace outside the hub. Record the Neve fact in Constraints and add an acceptance criterion: a hub holding a local marketplace of the operator's is refused at start."
  - id: F39
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:296
    summary: "The host relay writes into paths the area controls, and D12's Bound names only what it reads and posts. Its log, state, lock, outbox, inbox and notice records stay under <hub>/_bolt/.lym/slack (hubgeom/slackgeom.go), and the log is opened with O_APPEND and O_CREATE, which follows a link (slackengine/start.go). An agent that links relay.log to ~/.bashrc gets log lines carrying its own strings appended to a host shell file. No decision binds any host-side process against following area-made links."
    fix: "In D12, move every file the host relay and the socket holder write (log, relay state, lock, outbox, inbox, bridge notices and posts) into the area state directory. Add one rule, in D3 or D7, for every host-side process that touches a hub with an area (supervisor, lym area stop, relay, socket holder, host creel daemons): it opens nothing under the hub or area root through a symlink (openat2 with RESOLVE_NO_SYMLINKS and RESOLVE_BENEATH), and writes there only where a decision names the file. State the forwarding file, which in-area code writes, as read that way. Add an acceptance criterion: with relay.log and the slack directory replaced by links to a file outside the hub, starting the relay leaves that file unchanged."
  - id: F40
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:299
    summary: "D12 splits Slack across the area boundary without placing its files. In-area code reads presence, peers and the token file in the denied credential directory, and the setup and channel maps in the ungranted state directory, so away mode, lym slack hook and every cross-machine post (creelcli/slackbridge.go SendReady) fail inside, against D12's own bridge claim. The host relay finds its hub's creel socket under the host temp root, not the area root. Another hub's socket holder reads maps at hub-derived paths, so it acknowledges and drops an area hub's inbound events, or routes by a map an agent plants there."
    fix: "In D12, name for each Slack record (token, presence, peers, setup, channel map, bridge map, forwarding) where it lives for an area hub and who reads and writes it from which side. In-area readers either get a supervisor verb or a read grant on a directory holding no secret; the creel daemon's send check must not read the bot token. State that the host relay takes its hub's creel socket path and shortname from the area registry, and that every hub's socket holder reads an area hub's maps from its area state directory, found through the registry. Add acceptance criteria: a cross-machine post and lym slack away work from a pane of an area hub, and an operator message reaches an area hub's orch while another hub's relay holds the socket."
  - id: F41
    severity: BLOCKING
    class: consistency
    location: _lym/discussion/decision-record.md:342
    summary: "D15 departs from the declared PATTERN-closed-area text, only the supervisor decides a connect, by derived paths and the area file only areaengine reads. Under barrier apparmor the kernel profile decides every connect, the supervisor decides none, and the profile admits a creel-*.sock pattern in place of derived paths."
    fix: "In PATTERN changes, reword the PATTERN-closed-area bullet so it holds under both barriers within the rule-text cap, for example: only the area's barrier decides a connect, from paths lym renders and the area file only areaengine reads. In D15, state which of the entry's clauses the apparmor barrier meets and how."
  - id: F42
    severity: BLOCKING
    class: consistency
    location: _lym/discussion/decision-record.md:222
    summary: "D8 takes an area hub's repository name from the area file, but the PATTERN-agent-addressing background rule Across hubs gives authority by the repository name read from the hub's origin remote. The task declares no change to it. The PATTERN-child-processes background also says reaping a released process rests on no lym process being a subreaper, which the declared subreaper change falsifies, and the Notes do not name that sentence."
    fix: "In Notes for the plan writer, add that PATTERN-agent-addressing keeps its overview line and its Across hubs rule gains: for an area hub, the repository name its area-registry entry holds, never its origin remote. If the overview line must change, declare it under PATTERN changes. Add to the PATTERN-child-processes note that its Reap children paragraph on subreapers names the area supervisor, and that the supervisor reaps the orphans it adopts."
  - id: F43
    severity: MEDIUM
    class: design
    carries: F31
    location: _lym/discussion/decision-record.md:299
    summary: "D12's Bound and D16 still omit one Slack effect the host relay takes from area-written input. The relay renames and archives the orch channel and the bridge channels of every beam the hub's beam listing no longer names (slackengine/relay.go:689, keeper.go retire), and that listing comes from records agents write, so an agent can retire channels, ones shared with other machines included."
    fix: "In D12's Bound and D16's non-claims, add: an agent can make the relay rename and archive the orch and bridge channels of any beam by changing the hub's beam records."
prior:
  - id: F27
    status: open
    answer: The scope-name test handles the three named paths only while no process can leave its scope; clone3 with CLONE_INTO_CGROUP lets one leave. Carried as F33.
  - id: F28
    status: resolved
    answer: The ancestor walk and the direct-child rule close the nested hub. The separate path-argument route is F34.
  - id: F29
    status: resolved
    answer: The area registry takes the receive path off area-written files, and the different placement is sound.
  - id: F30
    status: resolved
  - id: F31
    status: open
    answer: The listed effects were added, but channel retirement from area-written beam records is still missing. Carried as F43.
  - id: F32
    status: resolved
    answer: D15 now holds no other hub's entry, which agrees with D6's grants as written.
---

Three claims the round-2 fixes rest on do not hold: that no process can leave its cgroup scope,
that a cwd walk covers every host verb, and that D6's grants let an area daemon reach other hubs.
The rest are exits the area leaves open or PATTERN texts the design departs from.
No kernel behaviour below was run here, since this review writes no probe program;
each finding names the source it rests on.

### [BLOCKING] A process can still leave its area scope, so D8's refusal fails open (carries F27)

**Location:** `_lym/discussion/decision-record.md:96`

**Issue:** clone(2) says `CLONE_INTO_CGROUP` takes a descriptor opened with `O_RDONLY` or `O_PATH`,
under the usual cgroup placement rules.
Those rules are a write check on `cgroup.procs`, and Landlock's hook list in `security/landlock/fs.c`
holds no `inode_permission` (read from a summary of the codebrowser page; the check in `cgroup.c` was not read).
On Neve `app.slice/cgroup.procs` is writable by the user, and a pane's scope sits directly under `app.slice`.
Go exposes the call as `SysProcAttr.UseCgroupFD`.

### [BLOCKING] D7's cwd walk misses path arguments and the exit hook

**Location:** `_lym/discussion/decision-record.md:193`

**Issue:** `lym reed window <dir>` skips the reed pre-run and, after opening the window,
calls `desktophook.AfterWindow` with the config of the worktree holding `<dir>`.
`window_command` is a machine key in `<hub>/_bolt/.lym/config/desktop.machine.yaml`, which agents write.
A refusal exits non-zero, and `NotifyExit` then arms the cwd-anchored trace sink,
which is the `.git` resolution D7 says the check runs ahead of.

### [BLOCKING] An area daemon cannot read what cross-hub mail needs

**Location:** `_lym/discussion/decision-record.md:150`

**Issue:** `landlock_append_fs_rule` keys a rule on the backing inode,
and `state` writes replace a record by `os.Rename` (`internal/fsx/fsx.go`).
`hubregistry.Lookup` opens the record's lock first, then each candidate hub's Bolt shortname file.
D6 grants neither, so a delivery from a hub without an area is refused as an unknown shortname,
and a post to one is refused the same way.

### [BLOCKING] `boot` builds a second Landlock domain

**Location:** `_lym/discussion/decision-record.md:76`

**Issue:** The supervisor is unrestricted, so a child it restricts is a new domain, however equal its rules.
The Landlock study records that two processes restricted separately cannot signal each other,
and the kernel documentation limits signals to the same or a nested domain.

### [BLOCKING] The attach client carries host secrets and the terminal into the area

**Location:** `_lym/discussion/decision-record.md:177`

**Issue:** D3 scrubs the server's environment, and nothing scrubs the client's.
tmux(1) says `update-environment` variables are copied into the session when an existing session is attached,
which also breaks D3's criterion that a pane holds no `SSH_AUTH_SOCK`.
`detach-client -E false` is enough to leave the Konsole window on a host shell in an area worktree.
`dev.tty.legacy_tiocsti` is 0 on Neve, so the risk there is reading and spoofing rather than injected keystrokes.

### [BLOCKING] A local marketplace inside the hub is host code the area writes

**Location:** `_lym/discussion/decision-record.md:57`

**Issue:** The loomyard hub is the hub this entry would first protect,
and its prime is the directory the operator's host sessions load the loomyard plugins from.

### [BLOCKING] The host relay writes through area-controlled paths

**Location:** `_lym/discussion/decision-record.md:296`

**Issue:** D12 moves the setup and the two maps out of the hub, and leaves the files the relay writes inside it.
D2 and D6 bound what the area can write, and nothing bounds where a host process's write lands after an area link.

### [BLOCKING] Slack's records are on the wrong side of the boundary

**Location:** `_lym/discussion/decision-record.md:299`

**Issue:** D12 routes only starting and ending the relay through the supervisor.
`RequireConfigured` reads the token file and the setup, `lym slack hook` reads and writes presence,
and `socketHolder.route` acknowledges an event that no readable map holds.
A map missing at the hub path reads as empty.

### [BLOCKING] The AppArmor barrier breaks the declared PATTERN-closed-area text

**Location:** `_lym/discussion/decision-record.md:342`

**Issue:** The task declares the entry and the design departs from it whenever `barrier: apparmor` is set.

### [BLOCKING] Two PATTERN backgrounds are contradicted without a declared change

**Location:** `_lym/discussion/decision-record.md:222`

**Issue:** The Notes name background edits for `PATTERN-config-layers`, `PATTERN-lym-files`
and the relay sentence of `PATTERN-child-processes`, and neither of these two.

### [MEDIUM] Channel retirement is missing from the relay's bound (carries F31)

**Location:** `_lym/discussion/decision-record.md:299`

**Issue:** The relay retires channels by the beam listing it reads from the hub.

## Focus answers

- Inside test: it refuses the three named peers while they stay in their scopes, and a plain host process passes.
  A process can leave what marks it (F33).
- Nested fake hub: the ancestor walk and the direct-child rule close that route.
  The refusal still misses path arguments and the exit hook (F34).
- Receive path: for a sending area hub no value that decides socket choice or trust comes from an area-written file.
  The host relay's own shortname still does (F40).
- `window`: not a defect.
  The area-chosen inputs are the canonical worktree path and the filtered title, which matches D13's Bound.
- Slack effects: one is missing (F43).
- D15 and D6: they agree as written, since no grant is made per other hub.
  F35's fix must keep it so.

## Held back by the incremental rule

These are new and either sit outside the changed hunks or fall below BLOCKING, so they are not findings.
The first two are BLOCKING in weight, and the parent should see them before the design is approved.

- D6 grants every right on `/dev/pts` (line 151).
  Every terminal of the user sits there (`crw-------`, owned by the user, on Neve),
  so an area process can open a host shell's or another hub's pane terminal, read its keystrokes and write to it.
  Landlock cannot tell one pty from another by path, so this needs another mechanism or a stated non-claim.
- D13's `lym area deploy` names no host-written source for the origin URL it clones or the parent branch it checks (line 313).
  Taken from hub records, both are agent-chosen, and the on-branch check then proves nothing.
- No decision removes a hub's area-registry entry when its area is turned off,
  so D8 keeps treating it as an area hub, against the Goal's "everything works as it does today" (MEDIUM).
- D3 installs the filter with `TSYNC` and `NEW_LISTENER` alone (line 85).
  `seccomp_set_mode_filter` returns `EINVAL` unless `TSYNC_ESRCH` is set too, as the probe sets it (LOW).
- D2's list of area-file keys lacks the deploy command D13 reads (LOW).
