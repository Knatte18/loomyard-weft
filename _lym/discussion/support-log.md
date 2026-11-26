# Support log — sandbox-closed-area-build

## Interview

### Turn 1 — the task and its inputs

The session ran autonomously with no advisors; every pick below is an auto-pick.
Read the board entry (a `motif` run), the board, the peers and the three studies under `docs/research/platform/`.
The entry's motif aim asks for a redesign around the seccomp-over-Landlock ranking, an AppArmor fallback,
a decision on what stays of the bwrap stages, and the pane memory cap kept under an area that refuses the bus.
No `Decision for sandbox-closed-area-build:` note exists on the board.
Task `reed-client-and-bus-guard` owns the shellguard bus rows, so this record leaves them out.

### Turn 2 — exploration

Two explore subagents mapped the pane memory cap and lym's bus use, and the config, shellguard and sandbox code.
The pane cap is made by the lym process that launches the pane, through `busctl`, with an uncapped fallback;
the gate's cap uses `systemd-run --user --scope` with an `RLIMIT_DATA` fallback.
No seccomp, Landlock or AppArmor code exists outside the research probes; lym builds with cgo; `golang.org/x/sys` is a direct dependency.
Own reads found that creel's `EXTERNAL` may inject into any strand, that lym's temp root is shared by every hub,
that the Slack relay reads tokens from lym's credential directory, and that Claude Code reads `CLAUDE_CONFIG_DIR` and `CLAUDE_CODE_TMPDIR`.

### Turn 3 — design picks

Each question, the recommended answer and the pick, all auto-picks:

- Mechanism: seccomp connect supervisor over Landlock ABI 8 (recommended), bwrap, or Landlock alone. Picked the supervisor (D1).
- Where the on-switch lives: a hub config module (as the entry said), or a per-user file outside the hub. Picked the file (D2): an agent writes every hub layer.
- System bus: on the allow list for hibernate, or off with a supervisor verb. Picked off (D5, D13).
- Shared temp root and tmux directory: grant write on them, or move the area's under a per-hub root. Picked the per-hub root (D6).
- Host-side lym over an area hub: allowed, or refused bar the area verbs and attach. Picked refused (D7).
- Cross-hub creel: drop it, give the supervisor its own creel daemon, or allow other hubs' sockets and bound what a daemon serves a peer from another area. Picked the last (D8).
- Memory cap: `RLIMIT_DATA` only, or a supervisor verb. Picked the verb (D9), since the operator requires the pane cap.
- Slack relay: inside with its tokens readable, or outside under the supervisor. Picked outside (D12).
- Egress proxy: keep stage 2's proxy, or drop it and keep its token rule. Picked drop (D11 and the scope).
- AppArmor: design it in full and build it only on a failed OSL-1046 probe. Picked (D14, D15).

### Turn 4 — lens review

Six forks returned reports: interactions, existing-paths, bound-breaker, walkthrough, board-scope and leanness; none returned without one.
Each finding, merged across lenses where they overlap:

- walkthrough 1 (BLOCKING), host tmux client runs an agent's `detach-client -E` command on the host: fixed, D6 runs the attach client under the area.
- interactions 1 and walkthrough 2 (BLOCKING), D8 refused every program post from dobby, the watchdog and the pusher: fixed, in-hub sender rules stay as today, and D16 names in-hub sender authenticity as a non-claim.
- interactions 2 (BLOCKING), inside detection failed under the AppArmor barrier: fixed, D3's inside test reads the hub's barrier, filter or profile label.
- existing-paths 1 and 4 (BLOCKING), account launch, env-module key and reed's scrub undid `CLAUDE_CONFIG_DIR`; user-setup read the host's `~/.claude`: fixed, D10 maps every account to an area directory, ignores the env key, sets the variables on the launch line and points every reader there.
- existing-paths 2 and interactions 6 (BLOCKING), creel found other hubs' sockets from the caller's own temp root: fixed, D6 records each hub's socket path in the machine record.
- bound-breaker 1 (BLOCKING), `deploy` on request runs any commit an agent pushed: fixed, D13 queues the deploy and only the operator's host-side `lym area deploy` runs it.
- bound-breaker 6 (MEDIUM), older daemons read a supervisor peer as `EXTERNAL`: fixed, D5 admits only hubs whose record carries the area-aware creel level.
- bound-breaker 7 and interactions 4 (MEDIUM), host-side inputs not fenced from write grants: fixed, D2's start check covers every host-side input and ancestor; D3's state directory holds nothing the area writes; D10's directories moved under the hub.
- walkthrough 4 (MEDIUM), the supervisor record was unreadable inside: fixed, D6 grants read on it.
- walkthrough 3 and interactions 3 (MEDIUM), D3's boot and restart contradicted each other: fixed, `lym area start` asks a live supervisor's `boot` verb, and reed boots through it from inside.
- walkthrough 5 and bound-breaker 4 (MEDIUM), D7 refused the relay: fixed, D7 exempts the supervisor-started relay and D12 bounds what it reads.
- interactions 5 (MEDIUM), in-area paths could not end the relay: fixed, D12's verb takes `stop` and the relay holds no `runproc` record.
- bound-breaker 5 (MEDIUM), an agent can claim `<shortname>:SLACK` inside the hub: rejected as a design change, since in-hub sender authenticity is no exit; D16 names it as a non-claim.
- existing-paths 3 (MEDIUM), the host boot read hub config: fixed, D3 builds tmux's argv from `PATH` or the area file.
- bound-breaker 8 (MEDIUM), other root-level steps ran before D7's check: fixed, D7's check runs first.
- walkthrough 6 (MEDIUM), git's credential route unstated: fixed, D11 names `lym area git-credential` and the `GIT_CONFIG_COUNT` override.
- bound-breaker 3 (MEDIUM), `RunMemoryCapped`'s early children escaped the scope: fixed, D9 caps a held child before `exec`.
- bound-breaker 2 (MEDIUM), any agent can call `hibernate` and `window`: rejected as a design change; D13 states the bound, since neither runs area code outside.
- walkthrough 7 (LOW), one unit prefix for every caller: fixed, D9's verb takes a kind.
- bound-breaker 11 (LOW), pidfd order: fixed in D9's bound.
- bound-breaker 9 (LOW), orphans outlived `stop`: fixed, D3 ends the tree with `runproc.EndProcess`.
- bound-breaker 10 (LOW), a loaded profile's paths are unreadable: fixed, D15 names the profile by a hash of its text.
- walkthrough 8 (LOW), the tip race: fixed, `lym area deploy` checks the commit is on the branch.
- walkthrough 9 (LOW), hubs added after the start: fixed, D5 reads the machine record at each decision.
- interactions 7 (LOW), the reverse creel spawn unstated: fixed in D3.
- interactions 8 (LOW), `PATTERN-run-records` miscited: fixed, the line is gone.
- board-scope 1 (MEDIUM), the Konsole step dropped silently: fixed, D16's README advice.
- board-scope 2 (LOW), the `PATTERN-config-layers` text conflicted with `config-keys-nested`: fixed, the entry's line stays and a note names the background sentence and the merge.
- board-scope 3 and leanness 6 (LOW), the `config-keys-nested` key-rename risk was settled: fixed, dropped.
- board-scope 4 (LOW), Claude Code deny rules unaddressed: fixed, out of scope, with the reason in Rejected alternatives.
- leanness 1 (MEDIUM), the D14 review bullet checked no diff: fixed, it now checks the note's split.
- leanness 2, 3 and 4 (LOW), constraints and scope restated decisions: fixed, cut.
- leanness 5 (LOW), auto-mode assumptions restated decisions: fixed, shortened to the assumptions.
- leanness 7 (LOW), a line-count tally: fixed, dropped.
- leanness 8 (LOW), D4's algorithm: kept, as the lens itself judged.
- leanness 9 (LOW), Turn 3 and the ledger repeat picks: rejected, the support log is not read by the plan writer.
- leanness, outside its lens, the `PATTERN-child-processes` text near the cap: fixed, measured under it.

## Rejected alternatives

- **bwrap as the area (the entry's stage 1).** Needs an AppArmor profile for user namespaces that OSL-1046 cannot load without IT.
- **Landlock ABI 8 alone.** Closes neither exposure, and `systemd-run --user` undoes it (measured in the Landlock study).
- **Taking the sockets away** (environment, modes, per-hub `TMUX_TMPDIR` without a filter). The owner sets modes back and a hard-coded bus address still starts a unit (measured).
- **A separate Unix user per hub.** Root per user, a sudo or polkit rule, hub ownership churn.
- **The `area` setting as a `configengine` module in the hub.** An agent edits the hub's config and turns the area off at the next start.
- **The system bus on the allow list for hibernate.** udisks and logind are reachable through it; a fixed verb is narrower.
- **The supervisor starting the hub's creel daemon to own cross-hub connects.** The daemon would be a sibling Landlock domain and its identity would rest on an exe check that `LD_PRELOAD` or ptrace from an ancestor defeats; bounding the receiving daemons is simpler.
- **Dropping cross-hub creel inside an area.** Receiving needs an outbound `confirm` too, so the hub would lose both directions.
- **Stage 2's egress proxy.** Landlock ABI 8 has no UDP control and the filter sees no `sendmsg`, so DNS and datagrams leave anyway; data leaving through an allowed host is not claimed either way.
- **Granting the Slack tokens inside the area.** They are a secret outside the area.
- **`RLIMIT_DATA` alone for the pane cap.** Loses the per-pane scope and its OOM-kill notice the operator requires.
- **Refusing an in-area sender that is in no pane.** dobby, the watchdog and the pusher post from no pane, and any forgeable program claim inside one domain gains nothing over today.
- **A `deploy` verb that runs on request.** An agent with the area's token can push any commit to the parent branch and ask for it.
- **Allowing every other hub's creel socket.** A daemon of an older binary reads the supervisor as `EXTERNAL`.
- **Claude Code deny rules in shuttle's settings as a second line.** The area holds without them, and shellguard already renders PreToolUse guards; a further deny list is work no exposure here needs.
- **The Konsole boot step as a lym change.** Inside an area, user namespaces escape neither Landlock nor the filter; for a hub without one, the README advice is enough.
- **A `PATTERN-config-layers` line change.** The area file is no config module, and the line is near the cap and rewritten by `config-keys-nested`; a background sentence suffices.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): mechanism — seccomp connect supervisor over Landlock ABI 8 (D1).
- Q2 (resolved, auto-pick): on-switch outside the hub, read only by `areaengine` (D2).
- Q3 (resolved, auto-pick): system bus off the allow list; hibernate as a verb (D5, D13).
- Q4 (resolved, auto-pick): per-hub area root for temp and sockets (D6).
- Q5 (resolved, auto-pick): host-side lym refuses over an area hub (D7).
- Q6 (resolved, auto-pick): cross-hub creel bounded at every receiving daemon, admitted only for area-aware daemons; in-hub sender rules unchanged (D5, D8).
- Q7 (resolved, auto-pick): memory caps through a supervisor verb (D9).
- Q8 (resolved, auto-pick): Slack relay outside the area under the supervisor (D12).
- Q9 (resolved, auto-pick): egress proxy dropped, token kept (D11).
- Q10 (resolved, auto-pick): AppArmor fallback designed, built only on a failed OSL-1046 probe (D14, D15).
- Q11 (open, operator action): run the supervisor probe on OSL-1046 under Cortex XDR before stage 1 (D14).
