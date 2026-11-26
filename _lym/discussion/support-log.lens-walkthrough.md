```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. I ran each decision on one concrete case. Two mechanisms defeat their own purpose (D1 and D8), and several more have a step whose input does not match what the step before it produces.

**Findings**

1. **BLOCKING, D1, D3, D6 and D13: the operator's tmux client outside the area runs whatever an agent tells it to.**
   - Case: the operator attaches from a host Konsole, through `lym reed attach` or D13's `window` verb. An agent in a pane connects to the hub's own tmux socket, which D5 allows, and runs `tmux detach-client -E '<cmd>' -t <operator's client>`.
   - The tmux man page says: "With -E, run shell-command to replace the client." The client is a host process outside the Landlock domain and the filter, so `<cmd>` runs on the host. `-P` also sends SIGHUP to the client's parent, the operator's shell.
   - The exit is the server-to-client path, which the record never considers. The supervisor sees no connect, and shellguard's raw-tmux row is only a guardrail.
   - The design needs a host attach that cannot be made to exec. One option is to run the attach client itself under the area's restrictions.
   - Evidence: `/usr/share/man/man1/tmux.1.gz`, the `detach-client` entry.

2. **BLOCKING, D8: refusing an in-area sender that is in no pane refuses every program post.**
   - Case: dobby, started detached inside the area, posts as `<shortname>:LOOM`. The reed watchdog posts the OOM notice that D9 relies on.
   - `internal/creelengine/doc.go` says "A process in no known pane is EXTERNAL", and through the Go client such a process "can post as a program". dobby, the watchdog, the pusher and other detached lym processes post that way.
   - D8 refuses an in-area sender in no pane, so all of these posts fail. Run mail and OOM notices stop inside an area.
   - D8 has to tell lym's own detached processes from an agent's double-fork. Doing that without a forgeable claim is the open design point.

3. **MEDIUM, D3 contradicts itself on a boot from outside.**
   - Case 1: area on, no supervisor, and the operator runs `lym reed attach` from a host shell. Bullet 4 says reed's spawn "goes through `lym area start`". The same bullet says reed refuses "with the area on and no live supervisor". The end state is undefined.
   - Case 2: the server has exited while dobby lives on. The supervisor lingers, so `lym area start` refuses ("a running supervisor for the hub"). Reed's refusal names `lym area start` as the way forward, but only `lym area stop` gets out of this state.

4. **MEDIUM, D3 and D8 need the supervisor record, but D6 grants no read on it.**
   - Case: the in-area creel daemon applies D8's rules, which need the supervisor's pid. That pid is in the record `lym area start` writes to the per-user lym state directory.
   - D6 grants read there only on the machine's hub record. The read fails, so the inside check and D8's refusals cannot work from inside.

5. **MEDIUM, D12 contradicts D7.**
   - Case: an in-area `EnsureRelay` asks the `slack-relay` verb, and the supervisor starts the relay on the host.
   - The relay is a `lym` verb whose cwd resolves to the hub, running outside the area. D7's root hook refuses every such verb except `lym area`, `lym reed attach`, version and help, so the relay never starts.
   - The relay is also host lym reading agent-writable hub files and state. This is the exposure D7 exists to close, and the record's bound for it rests on an unchecked claim that those files name no executable.

6. **MEDIUM, D11 never says how git gets pointed at the token file.**
   - Case: the background pusher pushes over HTTPS from inside. git takes its credential helper from `~/.gitconfig`, which D6 grants read, and on a `gh`-login setup that helper is `gh`. `gh` then reads `~/.config/gh`, which is denied, so the push fails.
   - "git's HTTPS credential reads only the area's token file" names the end state but not the route that overrides the host's helper.

7. **LOW, D9: the verb fixes the unit name `lym-pane-<pid>` for every caller.**
   - Case: `RunMemoryCapped` for a gate command. `proc.CapProcessMemory` takes a unit parameter, but D9 says the verb takes no unit name.
   - So gate processes land in `lym-pane-<pid>.scope`. The OOM watch skips any cgroup that is not a `lym-pane-` scope, so it may now read gate scopes as pane scopes.

8. **LOW, D13 deploy:** a "commit is that tip" check refuses whenever another landing reaches origin between the Land row's pull and the supervisor's fetch. Evidence: the pull-then-deploy order in `internal/battenshed/land.go`.

9. **LOW, D5:** a hub added to the machine record after the start is missing from the derived creel list until a restart. The record is silent on this.

**Decision walkthroughs**

| Decision | Case | Purpose |
|---|---|---|
| D1 | operator attached from the host, agent runs `detach-client -E` | not met (finding 1) |
| D2 | agent edits hub config to turn the area off | met: the file is outside every write grant |
| D3 | host `lym reed attach` with no supervisor; server exited while dobby lives | not met (findings 3 and 4) |
| D4 | connect to the bus, the swap race, a closed listener | met |
| D5 | pane connects to the session bus, own tmux, an added socket | met |
| D6 | tmux socket path `/tmp/lyma-1000-<8>/tmux-1000/<key>` | met: about 92 bytes, under 108 |
| D7 | host `lym board list` | met |
| D8 | dobby posting as a program; agent posting `deliver` to another hub | program post broken (finding 2); cross-hub bound met |
| D9 | pane cap through the verb; gate run | met for panes; unit naming (finding 7) |
| D10 | Claude session transcript inside the area | met |
| D11 | push over HTTPS | gap (finding 6) |
| D12 | relay start from inside | not met (finding 5) |
| D13 | window, hibernate, deploy | met, except the deploy tip race (finding 8) |
| D14 | probe run, then stage 1 | met |
| D15 | `barrier: apparmor` start | met, unmeasured |
| D16 | review only | not walked |

**Tool calls**

1. Bash, `internal/creelengine/doc.go` lines 40–60 plus a raw `tmux list-commands`. The guard refused the whole command, so neither read ran.
2. Bash, the `grep` of `internal/battenshed/land.go` for deploy and push order, the slackengine git imports, and the tmux man page for `detach-client`: my context had none of these. The land.go and man-page greps returned (the man page only up to `-t`); the slackengine grep printed nothing.
3. Bash, the rest of the `detach-client` man-page entry and the pull lines of `land.go`: confirms `-E` and `-P`, and the pull-then-deploy order.
