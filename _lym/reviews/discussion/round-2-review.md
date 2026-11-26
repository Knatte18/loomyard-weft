---
verdict: BLOCKING
findings:
  - id: F27
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:90
    summary: "D3's inside test, which D8's cross-hub refusal rests on, fails open. A peer that carries the area's filter but whose ancestry no longer reaches a supervisor or attach client the judging daemon can read is treated as a host process, so it gets every verb and posts as EXTERNAL into another hub. Three paths reach that state: an orphan of the attach client's detach-client -E command, which reparents to systemd --user and sends on an inherited connected creel socket; survivors of a dead supervisor holding already-open connections; and every peer of an area hub started after the judging area, whose supervisor record D6's per-file grant (line 144, fixed at landlock_restrict_self) cannot cover."
    fix: "In D3, replace the ancestry prong of the inside test with cgroup membership, which no area process can leave because /sys is read-only inside. Name every scope an area uses with the hub key: the supervisor's lym-area-<hub key>.scope, and D9's pane and run scopes under a lym-area-<hub key>- prefix. Have host-side lym reed attach ask the supervisor, before it restricts itself, to move it into a scope lym-area-<hub key>-attach-<pid>.scope, so the client's command and every orphan of it stay in that scope. State that a process is inside a hub's area when its /proc/<pid>/cgroup names such a scope, read with no supervisor record, and that D8 applies this test. Drop D6's read grant on every hub's supervisor record (line 144) and D3's sentence at line 67 if nothing else needs them. Add acceptance criteria: an orphan of the attach client's command posting on an inherited creel socket, a survivor of a killed supervisor, and a peer from an area hub started after the judging hub are each refused every verb but deliver and confirm."
  - id: F28
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:180
    summary: "D7 refuses by the hub a cwd resolves to, but lymcwd resolves a hub as the parent of the nearest .git root (lymcwd.go:121-132), and D3's boot (line 77) and D13's window (line 278) accept any worktree under the hub. An agent builds <hub>/x/wt with a .git file, a <hub>/x/_bolt/_lym and a reed.yaml naming its own tmux, then asks window for <hub>/x/wt. The host-side lym reed attach in Konsole resolves the hub <hub>/x, whose ServerName has no area file, so D7 does not refuse. The reed pre-run then loads the agent's reed.yaml (reedcli/cli.go:107), CheckTmuxFloor runs the agent's tmux on the host (attach.go:55), and the root pre-run's stencil seed pulls and commits over the agent's _bolt (stencilseed.go:52-59)."
    fix: "In D7, state the area check as a walk over the canonical cwd's ancestors: a verb refuses when any ancestor is a hub root whose ServerName has an area file, and the walk runs before any .git resolution and any other root step. In D3's boot and D13's window, accept only a worktree whose canonical parent is the hub root, never a deeper path. Add an acceptance criterion: with <hub>/x/wt holding a .git file and <hub>/x holding a _bolt/_lym and a reed.yaml whose tmux names a script, window and boot refuse it, and a host lym verb run in it refuses with the way forward, never running the script or any git."
  - id: F29
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:188
    summary: "D7 says cross-hub creel reads an area hub's machine-record entry alone, never its hub config, but a daemon receiving a deliver from an area hub resolves it through that hub's Bolt shortname file (hubregistry.Lookup, hubregistry.go:82; CreelGeometry, creelgeom.go:20) and stamps trust from RepositoryName, which reads the origin URL of the area prime's .git/config (repository.go:38, codebinding.go:71-73). Agents write both. Setting the origin to a repository the receiving hub lists in trusted_hubs makes their orch's mail arrive as that other repository's trusted orch (receive.go:294-304). That is authority outside the area, beyond D8 and D16's bound of trust in the area hub itself."
    fix: "In D8, state that a daemon receiving from an area hub takes that hub's shortname, socket path and repository name only from host-written data and never from the area hub's Bolt shortname file or prime .git/config. Give the area file a repository key the operator writes, and have lym area start copy it and the shortname into the hub's machine-record entry. Correct D7's sentence at line 188 to match. Add an acceptance criterion: an area hub whose prime origin is rewritten to a repository the receiving hub trusts has its orch's mail stamped under the area file's repository, untrusted unless that repository is listed."
  - id: F30
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:285
    summary: "D13's bound says the only area-written code a verb runs on the host is the landed commit's, but window takes a title from the area, and termwindow passes it into the host Konsole's command line as -p tabtitle=<title> (termwindow.go:96). Konsole's -p takes property=value, and the installed libkonsoleprivate carries Konsole::ProfileCommandParser::parse, which parses ;-separated property assignments. A title such as x;Environment=LD_PRELOAD=<hub>/a.so could then set host-side profile properties of the window's shell. The split is unmeasured, but the record bounds the title nowhere."
    fix: "In D13's window bullet and Bound, state that window takes no title from the area: the supervisor builds the title itself from the worktree's directory name, keeping only letters, digits, dot, underscore and hyphen. State that nothing else from the area reaches Konsole's arguments but the canonical worktree path checked per F28. Add an acceptance criterion: a window request whose worktree name holds ; or = opens a window whose Konsole arguments carry neither character in the title."
  - id: F31
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    carries: F17
    location: _lym/discussion/decision-record.md:269
    summary: "D12's bound says an agent's queued posts reach only the channel and operator the setup names, but the host relay also acts on area-written input in other ways. channelKeeper.create makes a public Slack channel per orch, named from the board project agents write, and invites the operator (keeper.go:146-170). Area-posted cross-machine creel mail makes bridge channels and invites other machines' bots (bridgechannel.go:54,91; bridgeout.go:287-288), so area mail reaches other machines' orchs under this machine's identity. D16 names none of it."
    fix: "In D12's Bound, name every Slack effect the host relay takes from area-written input: the per-orch public channel and its name from the board project, the operator invite, and the cross-machine bridge channels and peer-bot invites. Either fix each input host-side (channel names from the setup in the area state directory, and cross-machine bridges off for an area hub) or add each to D16's non-claims. The non-claim must state that an area hub's orch mail reaches other machines' orchs with this machine's standing."
  - id: F32
    severity: MEDIUM
    class: design
    origin: lens:interactions
    carries: F23
    location: _lym/discussion/decision-record.md:304
    summary: "D15 renders the profile from the area file's lists and the Landlock paths and claims its text does not change when a hub records itself. But D6's Landlock paths include a read grant on every hub's supervisor record (line 144), so each new area hub changes every AppArmor hub's rendered text and hash, and the start refuses until root reloads the profile."
    fix: "In D15, render every per-hub Landlock entry as one path pattern, as the creel sockets already are, or leave per-hub entries out of the rendered text, so a new area hub changes no hub's profile hash. If F27's fix drops the supervisor-record grant, state that D15's text then holds no per-hub entry. Otherwise state the root reload each new area hub costs."
prior:
  - id: F1
    status: resolved
    answer: D6 and D7 now state that the area hub's host-side attach reads no hub config or state before it restricts itself; the nested-hub route around it is the new F28.
  - id: F2
    status: resolved
    answer: The supervisor socket moved to the read-only state directory with a SO_PEERCRED check. The rebuttal on the creel half is accepted, since D8 and D16 bound it.
  - id: F3
    status: resolved
  - id: F4
    status: resolved
  - id: F5
    status: resolved
  - id: F6
    status: resolved
  - id: F7
    status: resolved
  - id: F8
    status: resolved
  - id: F9
    status: resolved
    answer: The rebuttal is accepted. The exit-before-read and attach-client cases are handled, and refusing every filtered peer would hit host processes. The fail-open that remains through orphans, survivors and later hubs is the new F27.
  - id: F10
    status: resolved
    answer: D6 grants the read, but a per-file grant cannot cover a hub started later; that gap is part of F27.
  - id: F11
    status: resolved
  - id: F12
    status: resolved
  - id: F13
    status: resolved
  - id: F14
    status: resolved
    answer: Rebuttal accepted; barrier plus descent refuses host-side children and attach clients alike.
  - id: F15
    status: resolved
  - id: F16
    status: resolved
  - id: F17
    status: open
    answer: The setup moved out of the hub, but the new bound overstates what the relay does with area-written input; carried as F31.
  - id: F18
    status: resolved
  - id: F19
    status: resolved
    answer: Rebuttal accepted; scopes name the area's processes whether the supervisor lives or not.
  - id: F20
    status: resolved
    answer: Rebuttal accepted; the cgroup filter bounds the signalled pids.
  - id: F21
    status: resolved
  - id: F22
    status: resolved
  - id: F23
    status: open
    answer: Creel sockets became a pattern, but the per-hub supervisor-record grant still changes the rendered text; carried as F32.
  - id: F24
    status: resolved
  - id: F25
    status: resolved
  - id: F26
    status: resolved
    answer: Rebuttal accepted; the host plugin-cache grant is gone and D11 cites gitcredentials(7).
---

The fixes closed most of round 1, and the round's focus led to four new holes, each on a changed hunk.
Three let area-written input decide what a host process does: a nested fake hub, a spoofed origin URL and an unbounded window title.
The fourth is that the inside test fails open.
All four lenses found the nested hub, the origin spoof and the fail-open test independently, and two found the title.

### [BLOCKING] D3's inside test fails open, so D8's cross-hub refusal misses filtered orphans, survivors and later hubs

**Location:** `_lym/discussion/decision-record.md:90`, with D6:144 and D8:201-203.

**Issue:** The test needs the barrier and ancestry to a recorded supervisor or attach client.
A process that fails it counts as a host process, and D8 then serves it every verb.
Three paths leave a filter-carrying area process without readable ancestry:

- The `detach-client -E` command of the operator's attach client forks a child that inherits a connected creel socket. The child reparents to `systemd --user`, since the client is no descendant of the subreaper (line 172).
- A supervisor dies, and its survivors keep connections that are already open (line 121).
- An area hub starts after the judging hub, whose fixed ruleset grants no read on the later hub's record (line 144).

Each then posts as `EXTERNAL` into another hub, which D8's own rationale calls an act outside the area.
Cgroup membership survives all three paths, and the record already relies on it in D3's stop.

### [BLOCKING] A nested fake hub under the area hub escapes D7, and `window` delivers it to host lym

**Location:** `_lym/discussion/decision-record.md:180`, with D3:77 and D13:278.

**Issue:** `lymcwd` takes the hub as `filepath.Dir` of the nearest `.git` root (`internal/lymcwd/lymcwd.go:121-132`).
So `<hub>/x/wt` resolves to the hub `<hub>/x`, which has no area file.
`window` and `boot` accept that path because it lies under the hub.
The host attach then runs the agent's `tmux` and the stencil seed's git on the host.
Each lens built the same counterexample from the code in its context.

### [BLOCKING] Cross-hub creel takes an area hub's identity and trust from area-written files

**Location:** `_lym/discussion/decision-record.md:188`, with D8:201-212.

**Issue:** D7's sentence is false about today's receive path, `Deliver` → `Lookup` / `CreelGeometry` / `RepositoryName` → `trustedSender` (`internal/creelengine/receive.go:133-178, 294-304`).
The area's agents can claim a repository another hub trusts, which goes past the bound the record concedes.

### [BLOCKING] D13's `window` passes an unbounded area-chosen title into the host's Konsole command line

**Location:** `_lym/discussion/decision-record.md:285`, the Bound line, with the `window` bullet at 278.

**Issue:** `konsoleArguments` builds `-p tabtitle=<title>` (`internal/termwindow/termwindow.go:96`).
Konsole parses `-p` values with `ProfileCommandParser::parse`, a symbol in the installed `libkonsoleprivate.so.25.12.3`, and that parser reads `;`-separated property assignments.
Whether an injected property reaches the shell is unmeasured.
The fix costs nothing, and the Bound's claim cannot stand while the title is open.

### [MEDIUM] D12's relay bound overstates what an area can make the relay do (carries F17)

**Location:** `_lym/discussion/decision-record.md:269`.

**Issue:** The relay creates channels, invites users and bridges to other machines from area-written input (`internal/slackengine/keeper.go`, `bridgechannel.go`, `bridgeout.go`, read by the bound-breaker lens).
D12's bound and D16's non-claims name only queued posts to the setup's channel.

### [MEDIUM] D15's profile text changes with every new area hub (carries F23)

**Location:** `_lym/discussion/decision-record.md:304`.

**Issue:** The per-hub supervisor-record grant enters the rendered Landlock paths.
That contradicts D15's no-reload claim, which the creel-socket pattern was meant to secure.

## Held back by the incremental rule

The lenses also raised these below BLOCKING.
They are new and sit outside what this incremental round may raise, so they are not findings here.
None is a false positive, and each is worth the writer's attention when it next touches these lines:

- D2's fence (lines 55-58) omits host-side inputs other decisions name: `~/.gitconfig` and `~/.config/git`, the machine hub record and `machine.json`, the Claude Code launcher and versions, the local marketplace directories, and login and autostart paths (interactions, bound-breaker; MEDIUM).
- An older binary's `hubregistry.Record` rewrite may drop the per-entry socket path and area-aware level D5 and D6 add (interactions; MEDIUM).
- D9's supervisor-made run scope has no holder, so it may be collected before `RunMemoryCapped` reads `memory.events` after an OOM kill (walkthrough; MEDIUM).
- D11's `GIT_CONFIG_COUNT` override is missing from D3's list of server variables (walkthrough; LOW).
- D8's `SCM_PIDFD` and fdinfo `Pid: -1` behaviour, and D3's claim that stopping a scope ends every process in it, are stated without a pin (bound-breaker, handler; LOW).
