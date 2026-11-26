```yaml
lens: bound-breaker
outcome: reported
```

Task: review the decision record through the bound-breaker lens. I attacked each bound, guard and claim and looked for counterexamples.

**Claims attacked**

| Claim | Result |
|---|---|
| D1: Landlock plus the filter close the bus and other hubs' tmux | held |
| D2: the start check fences write grants from host-side inputs | broken (finding 6) |
| D3: the inside test (barrier plus ancestry to the supervisor or a recorded attach client) | broken (finding 1) |
| D3: `boot` takes only a worktree under the hub | broken (finding 2) |
| D3: `lym area stop` signals only pids in the area's scopes | held. Group signals reach only one session's group, and an area process cannot join a host process group. |
| D3/D6: host callers check the supervisor's `SO_PEERCRED` pid against the read-only record | held |
| D4: the filter fails closed when the supervisor dies | held for new connects. Already-open connections survive, which feeds finding 1. |
| D4/D16: TCP and UDP are open, and that is stated as a non-claim | held |
| D5: a derived creel path is held to the creel socket's name and place | held. Area agents cannot write the machine record. |
| D6: the supervisor socket cannot be unlinked or replaced from inside | held |
| D6: `PR_SET_PTRACER` makes the client's connects decidable under Yama 1 | held for the client itself; the client's forks lead into finding 1 |
| D7: host lym reads no area hub config and runs no git over an area hub | broken (findings 2 and 5) |
| D7: cross-hub creel reads only the machine-record entry | broken (finding 5) |
| D8: an agent cannot post as `EXTERNAL` to another hub | broken (finding 1) |
| D8: the kernel refuses a forged `SCM_CREDENTIALS` pid | held |
| D8: the pidfd liveness check | unpinned (finding 8) |
| D9: `cap-memory` refuses host-side children and attach clients | held |
| D10: hooks run only in the area's own sessions | held |
| D11: the token rule | held |
| D12: agent posts reach only the channel and operator the setup names | broken (finding 4) |
| D13: `window` runs no area code outside the area | broken (findings 2 and 3) |
| D13: `deploy` runs only the landed commit, on the operator's command | held, as stated |
| D15: the profile text does not change when a hub records itself | broken (finding 7) |

**Findings**

1. **BLOCKING, design.** `decision-record.md:90` (D3's inside test, used by D8 at lines 201–203)
   - Summary: the inside test fails open. A peer that carries the filter but whose ancestry does not reach a supervisor or attach client the judging daemon knows is treated as a host process. It gets every verb and posts as `EXTERNAL` into another hub.
   - Counterexample (a): a client turned command by `detach-client -E` connects to hub C's creel socket, which D5 allows. It forks a child that inherits the connected fd, then exits. The child is reparented to `systemd --user` (the client is not under the subreaper, D6:171–172). Its `SCM_CREDENTIALS` names it, its ancestry reaches nothing in the record, so it is `EXTERNAL` at C.
   - Counterexample (b): a pane process passes a connected socket by `SCM_RIGHTS` over `sendmsg` to a named datagram socket, which D4:118 lets through, into such an orphan.
   - Counterexample (c): D6:144 grants read on "every hub's supervisor record" in a ruleset fixed at start. An in-area daemon of hub A cannot read the record of an area hub B started later, so B's agents do not test as inside at A.
   - Counterexample (d): after a supervisor crash (D4:121), survivors keep their already-open creel connections while their ancestry now reaches `systemd --user`.
   - Evidence: `decision-record.md:90-92,118,121,144,169-172,201-203`; probe `serveTestConnection` reads the sender's pid per message (`seccomp-connect-probe/main.go:516-524`).

2. **BLOCKING, design.** `decision-record.md:278` (D13 `window`; also D3's `boot` at line 77 and D7 at line 180)
   - Summary: "a worktree under the hub, checked by canonical path" admits a nested fake hub. Host lym resolves the hub as the worktree root's parent, so it is not the area hub. No area file matches, D7 does not refuse, and host lym reads agent config and runs git over agent-written repositories.
   - Counterexample:
     - An agent builds `<hub>/x/wt/.git`, `<hub>/x/_bolt/_lym` and `<hub>/x/_lym/config/reed.yaml` with `tmux: <hub>/evil.sh`, then calls `window` with `<hub>/x/wt`.
     - Konsole's host `lym reed attach` resolves `hubPath = <hub>/x` (`lymcwd.go:126`). `ServerName(<hub>/x)` (`server.go:47`) names no area file, so the verb is not refused.
     - The reed pre-run loads the agent's `reed.yaml` (`reedcli/cli.go:107`), and `CheckTmuxFloor(c.eng.TmuxPath())` runs `evil.sh` (`attach.go:55`).
     - The root pre-run's `seedStencils` pulls and commits in `<hub>/x/_bolt` (`stencilseed.go:52-59`), whose `.git/config` the agent wrote.
   - Fix direction: accept only a direct child of the hub root, and have D7 refuse any cwd that lies under an area hub's root.

3. **BLOCKING, design.** `decision-record.md:278` (D13 `window` takes "a title" from the area)
   - Summary: the title is unbounded. It reaches Konsole as `-p tabtitle=<title>` (`termwindow.go:96`).
   - Evidence: Konsole's `-p` takes `<property=value>` (`konsole --help`), and the installed `libkonsoleprivate.so.25.12.3` exports `Konsole::ProfileCommandParser::parse`, which splits property assignments on `;`. A title such as `x;Environment=LD_PRELOAD=<hub>/a.so` or `x;Command=…` could therefore set host-side profile properties of the window.
   - Caveat: the effect itself is unmeasured. D13's bound ("neither runs area code outside the area") states no limit on the title.

4. **BLOCKING, design.** `decision-record.md:269` (D12's bound: posts "reach only the channel and operator the setup names")
   - Summary: the host relay acts on agent-written input beyond posting to the setup's channel.
   - `channelKeeper.create` makes a public Slack channel per orch, named from the board project, which agents write through `k.hub.Project`, and invites the operator (`keeper.go:146-170`).
   - Cross-machine creel mail an agent posts makes bridge channels and invites other machines' bots (`bridgechannel.go:54,91`; `bridgeout.go:287-288`). Area mail then reaches other machines' orchs through the operator's bot, outside the area, and D16 does not list it as a non-claim.

5. **BLOCKING, design.** `decision-record.md:188` (D7: "Cross-hub creel reads an area hub's machine-record entry alone, its socket path, never its hub config")
   - Summary: a host daemon receiving a `deliver` from an area hub reads and runs git over that hub's agent-written state, and takes trust from it.
   - `Deliver` → `hubs.Lookup` → `hubregistry.Lookup` reads each hub's Bolt shortname (`hubregistry.go:82`).
   - `confirmWithSender` → `hubs.Geometry` → `CreelGeometry` derives the socket from the Bolt shortname, not from the entry's path (`receive.go:260`; `creelgeom.go:20,31`).
   - `sendingRepository` → `RepositoryName` → `HubOriginURL` → `gitrepo.New(prime).RemoteURL("origin")` runs git over the area prime's agent-written `.git/config` (`receive.go:176`; `repository.go:38`; `codebinding.go:71-73`).
   - Agents can therefore set their origin to a repository the receiving hub lists in `trusted_hubs`, and their orch's mail is stamped trusted (`receive.go:294-304`). That is more than D8's "trust in an area hub", because it impersonates another repository.

6. **MEDIUM, design.** `decision-record.md:55-58` (D2's fence list)
   - Summary: the start check omits host-side inputs that D6 grants read on and that the host itself runs or reads.
   - Not listed: `~/.gitconfig` and `~/.config/git` (host git), the machine hub record and `machine.json` (read by host daemons and D5), the Claude Code launcher and versions (host executes them), and the local marketplace directories D10 reads (host Claude installs from them).
   - Login and autostart paths (`~/.profile`, `~/.config/systemd/user`, `~/.config/autostart`) are not listed either.
   - Result: a `write` entry on any of them passes the check and gives a persistence exit against the Goal.

7. **MEDIUM, consistency.** `decision-record.md:304-307` (D15)
   - Summary: the profile renders "the Landlock paths", which by D6:144 include every hub's supervisor record. A new area hub changes the rendered text and so the hash, and the start refuses until root reloads the profile. That contradicts "its text does not change when a hub records itself".

8. **LOW, design.** `decision-record.md:195-198` (D8)
   - Summary: `SO_PASSPIDFD`/`SCM_PIDFD` (Linux 6.5) and the fdinfo `Pid: -1` liveness rule are tool facts stated with no measurement, test or document cited, unlike the `SO_PEERCRED` row next to them.

**Tool calls**

1. `grep` for `CreateChannel`, `Invite` and `Archive` callers in `slackengine`: the manifest held only the API's method list, not what drives the calls.
2. `sed` of `keeper.go:135-170` and `bridgeout.go:270-295`: to see what input names and triggers the channel create and the peer-bot invite.
