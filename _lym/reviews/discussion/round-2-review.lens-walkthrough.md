```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on one concrete case.

**Decisions walked**

- D1, case: an area hub boots its server. Purpose met.
- D2, case: an area file keyed by `ServerName(hubPath)`. Purpose met for the hub's own path. A nested path under the hub derives a different key; that case is filed under D7.
- D3, case: a `boot` for a cold worktree and `lym area stop` after the supervisor is killed. Purpose met, but the inside test is broken by the D8 cases below.
- D4, case: a pane connects to the session bus, then to its own tmux socket. Purpose met.
- D5, case: a pane connects to another hub's creel socket through the machine-record entry. Purpose met.
- D6, case: the operator runs `lym reed attach` from a host Konsole. Purpose met for the hub's own worktrees. It breaks through D7's nested-hub case.
- D7, case: the `window` verb is handed `<hub>/evil/wt`. **Purpose broken.**
- D8, case: area hub A delivers to host hub B, which lists repository X as trusted. **Purpose broken.**
- D9, case: `RunMemoryCapped` whose command is OOM-killed at the cap. **Purpose partly missed.**
- D10, case: a Claude session under the default account inside an area. Purpose met.
- D11, case: `git push` over HTTPS from a pane. Purpose met, with one gap in where the environment is set.
- D12, case: `slack-relay start` from inside, with the relay double-forking. Purpose met: the supervisor is a subreaper, so the relay is reparented to it and stays in its scope.
- D13, case: an agent calls `window` with a crafted title. **Purpose broken.**
- D13 `hibernate`, case: `lym hibernate request` inside an area. Purpose met: the request is per hub (`internal/hibernatecli/cli.go:24`).
- D14, D15, D16: no runnable mechanism to walk.

**Findings**

1. **BLOCKING, design — `_lym/discussion/decision-record.md:180` (D7), with D13 `window` at :278 and D3 `boot` at :77.**
   D7's refusal goes by the hub the cwd resolves to, and `window` accepts any worktree "under the hub". An agent can therefore build a fake hub nested inside its own hub and get the host to run its binary.
   - Step 1: the agent creates `<hub>/evil/wt` with a `.git` file, `<hub>/evil/_bolt/_lym`, and `<hub>/evil/wt/_lym/config/reed.yaml` whose `tmux` names a script.
   - Step 2: it calls `window` with that path. The canonical path lies under the hub, so the check passes, and Konsole runs the host's `lym reed attach` there.
   - Step 3: `lymcwd.Resolve` takes `hubPath = filepath.Dir(workTreeRoot)`, which is `<hub>/evil` (`internal/lymcwd/lymcwd.go:126`). `ServerName` of that path has no area file, so D7 does not refuse.
   - Step 4: the reed pre-run's `wireLocation` loads that `reed.yaml` (`internal/reedcli/cli.go:107`). `CheckTmuxFloor(c.eng.TmuxPath())` then runs the agent's script on the host (`internal/reedcli/attach.go:55`).
   - Step 5: the root pre-run's stencil seed also does a Bolt pull and commit over the agent's `_bolt` (`cmd/lym/stencilseed.go:52-59`).

   Fix: D7 refuses whenever the cwd's canonical path lies at or under any area hub's root, not only when the resolved hub has an area. D3's `boot` and D13's `window` accept only a direct child of the hub root. Add an acceptance criterion with a nested fake hub.

2. **BLOCKING, design — `_lym/discussion/decision-record.md:188` (D7, "Cross-hub creel reads an area hub's machine-record entry alone … never its hub config") and D8 :209-212.**
   A daemon receiving mail from an area hub decides trust from that hub's own repository name, which the area's agents write. They can borrow the trust of any repository another hub lists.
   - Step 1: B's `Deliver` resolves the sending hub with `hubs.Lookup`, which reads A's Bolt shortname (`internal/hubregistry/hubregistry.go:82`).
   - Step 2: `sendingRepository` calls `RepositoryName`, which reads the origin URL from A's prime `.git/config` (`internal/fabricengine/codebinding.go:71-73`, `internal/hubregistry/repository.go:37`). That file is agent-writable.
   - Step 3: `trustedSender` stamps A's orch mail as trusted when that name is in B's `trusted_hubs` (`internal/creelengine/receive.go:294-304`).
   - End state: A's agents set their origin to X and their mail reaches B as a trusted orch, though B never trusted A. D8 and D16 only concede "trust in an area hub", which this exceeds.

   Fix: D8 takes an area hub's repository name from a host-written source, such as the area file or the machine-record entry `lym area start` writes, and never from the area's `.git/config`. Add an acceptance criterion with a rewritten origin.

3. **BLOCKING, design — `_lym/discussion/decision-record.md:278` (D13 `window`).**
   The area chooses the window title, and termwindow passes it into Konsole as `-p tabtitle=<title>` (`internal/termwindow/termwindow.go:96`). Konsole's `-p` takes `property=value` (`konsole --help`), and the installed `libkonsoleprivate` carries `Konsole::ProfileCommandParser::parse`. My understanding is that this parser splits the value on `;` into further profile properties, among them `Command` and `Environment`. If so, a title such as `x;Environment=LD_PRELOAD=<hub>/evil.so` reaches the host's shell. The `;` split is unmeasured here.
   - D13's bound says neither verb runs area code outside the area, but it does not bound the title at all.

   Fix: D13's `window` takes no title from the area (the supervisor derives it from the worktree name), or limits it to a fixed character set. Add an acceptance criterion with a title containing `;`.

4. **MEDIUM, design — `_lym/discussion/decision-record.md:220-224` (D9).**
   The supervisor-made scope has no holder process, so after an OOM kill the scope can be collected before `RunMemoryCapped` reads it.
   - Step 1: the command is OOM-killed and `RunMemoryCapped` wakes.
   - Step 2: today the scope outlives its command because `scopeStartScript`'s holder keeps it "until the caller has read them" (`internal/proc/memorycap_linux.go:49-51`). D9 replaces that script's marker with the verb's cgroup answer and does not mention the holder.
   - Step 3: the supervisor's transient scope for the held child empties when the tree dies, and systemd may remove it.
   - Step 4: `scopeHadOOMKill` then reads a missing `memory.events` and reports false (`internal/proc/memorycap_linux.go:207-209`). `treeKilled` is lost, and the run's report misses the kill at the cap.

   Fix: D9 states what keeps the run scope alive until its events are read, for example the held child's holder, or a `cap-memory` scope that the supervisor removes only on `kill-scope` or a release verb.

5. **LOW, consistency — `_lym/discussion/decision-record.md:253` (D11) against D3 :81-83.**
   D11 has "the area's launch environment" set `GIT_CONFIG_COUNT`, but D3's server-environment bullet lists the exact variables the supervisor removes and sets, and that list omits `GIT_CONFIG_*`.
   - Walkthrough: a pane's `git push` inherits the host's `~/.gitconfig` helpers, such as a `gh` or store helper. Their files are denied inside, so the push fails instead of using `lym area git-credential`.

   Fix: add the `GIT_CONFIG_COUNT` entries to D3's list of variables the supervisor sets.

**Tool calls**

1. Grepped `internal/reedcli/watchdog.go` for signal or kill calls: to check whether a watchdog in a separate Landlock domain needs to signal panes. It doesn't, so no finding.
2. Read `gitrepo.Repo.RemoteURL` in `internal/gitrepo/remote.go`: to check whether the receiving daemon spawns git over the area repository. It reads in-process with go-git, so finding 2 rests on the trust read, not on git running.
3. Listed `internal/hibernatecli` and grepped it and the hibernate engine: to walk D13's `hibernate` reach from inside an area. The request is per hub, so no finding.
