---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed-differently
    reason: "The supervisor-socket half is fixed as proposed. The creel half's proposed peer check cannot tell an agent's listener from the hub's daemon: both run inside the same area and pass D3's inside test, and the support log's Rejected alternatives already records that an exe check falls to LD_PRELOAD. Any agent can also drive the orch's pane through the hub's own tmux socket, which D5 allows. D8 and D16 now state the bound instead: a confirm from an area hub proves only that the mail came from inside that area, so another hub's trust in an area hub is trust in every agent of it. No replaced-creel-socket criterion is added, since nothing it could check is promised."
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed-differently
    reason: "The gone-or-unreadable pid rule and the pid read (SCM_PIDFD, plus a liveness check after the /proc reads) are made as proposed. The proposed refusal of any peer that carries a seccomp filter is not: host processes carry filters routinely (on Neve, chrome shows Seccomp_filters 1), and every daemon, area or not, would change behaviour, against the Goal's 'With the area off, everything works as it does today'. Instead the supervisor records each attach client, and D3's inside test counts a process whose ancestry reaches a recorded attach client, so other hubs' daemons refuse the attach client's command."
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed-differently
    reason: "D3's inside test now also counts the operator's recorded attach clients (F9). Requiring the full inside test would let an agent cap the operator's attach client. D9 requires the pid to carry the hub's barrier and to descend from the supervisor, which refuses the supervisor's host-side children and the attach clients alike."
  - id: F15
    disposition: fixed
  - id: F16
    disposition: fixed
  - id: F17
    disposition: fixed
  - id: F18
    disposition: fixed
  - id: F19
    disposition: fixed-differently
    reason: "The proposal's way of finding the processes, by barrier plus a recorded cgroup or start time, cannot name the hub: a seccomp filter is not hub-specific, and an orphan's ancestry no longer reaches the dead supervisor. Instead, lym area start puts the supervisor in its own transient scope, and the record lists every scope cap-memory makes. lym area stop stops those scopes over the bus, which ends every process in them whether the supervisor is alive or dead. Area processes cannot leave the scopes, since /sys is read-only inside. The proposed acceptance criterion is added."
  - id: F20
    disposition: fixed-differently
    reason: "With a dead supervisor, the proposed inside test (barrier plus ancestry to the supervisor) fails for every area process, so stop could signal none of them. Cgroup membership names the area's processes in both cases: stop signals only a pid in the area's scope or a recorded scope, whatever the run records name. The D3 stop criterion carries it."
  - id: F21
    disposition: fixed
  - id: F22
    disposition: fixed
  - id: F23
    disposition: fixed
  - id: F24
    disposition: fixed
  - id: F25
    disposition: fixed
  - id: F26
    disposition: fixed-differently
    reason: "The plugin-cache write was observed and recorded: Claude Code writes .in_use/<pid> files under each plugin version during a session. What fails when that write is refused was not determined, since that needs Claude Code run under Landlock. Rather than keep a grant whose failure is unknown, D10 drops the host plugin-cache grant, and the area installs its own plugins, a local-directory marketplace being read from its path. D11's empty-helper reset cites gitcredentials(7) as proposed."
---

# Round 1 fixer report — discussion

Every change is in `_lym/discussion/decision-record.md`, plus a round entry in `_lym/discussion/support-log.md` that records the measurements.

## Changes

- F1: D6 gains an attach bullet. A host-side attach of an area hub reads no hub config or state before it restricts itself: it takes tmux from `PATH` or the area file, finds the socket from the area root, and skips `EnsureSession`, `Status` and the watchdog spawn. It then asks `boot` for its worktree and runs tmux, its floor check included, only after the restriction. D7 points at that bullet. New criterion: a hub `reed` tmux naming a script under the hub is never run.
- F2 (fixed differently, see frontmatter):
  - The supervisor's socket moves to the area state directory (D3, D6).
  - Every host-side verb caller checks `SO_PEERCRED` against the record's pid (D3).
  - New criterion: an area process cannot replace the supervisor's socket, and a wrong peer is refused.
  - D8's Bound and D16 state the creel trust bound.
- F3: D6 has attach call `prctl(PR_SET_PTRACER, <supervisor pid>)` before the filter, citing the kernel Yama documentation. Measured on Neve: without the declaration a sibling's `pidfd_getfd` and `/proc/<pid>/mem` fail with `EPERM`; with it both succeed, also after `exec`. The result is in the Constraints and the support log. D6 also says that processes the client's command forks fail closed.
- F4:
  - D6 gives the area its own `GOCACHE` and `GOMODCACHE` under `<hub>/.lym/area/` and denies write on the host's caches.
  - D2's start check lists the host caches.
  - D13's deploy uses the host's own caches.
  - The D6 criterion extends to writes into the host's caches.
- F5: D7 refuses over an area hub however the verb resolves it. Hub walks (`lym machine start`, the sweeps) skip area hubs with a notice. Cross-hub creel reads the machine-record entry only. New criterion for `lym machine start`.
- F6: D6 makes `internal/lymtemp` own the area root: 0700, created by the supervisor, and refused unless it is a real directory owned by the user. `TMPDIR` points at a lymtemp run directory under it. New criterion. A plan-writer note covers `PATTERN-lym-files`' background.
- F7: D3's `boot` builds the server environment: reed's scrub and strips are applied, the token, bus and agent variables are removed, the temp and cache variables are set, and the `-S` socket path matches attach's. New criterion.
- F8:
  - D3 orders `no_new_privs`, then Landlock `TSYNC`, then seccomp `TSYNC`, citing landlock_restrict_self(2).
  - The Constraints' cgo line is replaced with the measured result: in a cgo build, Landlock `TSYNC` carries `NoNewPrivs: 1` and the domain to every thread, and with Landlock first the call fails with `EPERM`.
  - The D1/D3 criterion checks every thread.
- F9 (fixed differently): D8 takes `SCM_PIDFD` beside `SCM_CREDENTIALS` and checks the pidfd's liveness after its `/proc` reads. It refuses a gone or unreadable sender all but `deliver`/`confirm`. Attach clients count as inside their hub's area through D3's widened inside test. New criterion.
- F10: D6 and D8 grant read on every hub's supervisor record.
- F11: D5 admits a derived creel path only with that hub's socket name, directly in a lym temp root or an area root. New criterion for an entry naming the bus.
- F12: the Goal now excludes TCP and UDP. D4's Bound and D16's non-claims name every TCP and UDP service, loopback and link-local included.
- F13: in D9, `cap-memory` records the scope and answers its cgroup path, and a new `kill-scope` verb writes `cgroup.kill` for recorded scopes. The kill is added to the D9 criterion.
- F14 (fixed differently): D9 requires the barrier plus descent from the supervisor. The criterion refuses the relay's pid.
- F15: D10 says that inside an area codexengine ignores `codex.yaml`'s `home`, makes no links into the operator's Codex home, and keeps `auth.json` and `sessions` in the area. New codex criterion.
- F16: D12 turns the relay's `runproc` self-registration off and has an area process end the relay only through the verb. The D12 criterion checks for no record. A plan-writer note covers `PATTERN-child-processes`' background.
- F17: D12 moves the setup (operator id, channel and bridge maps) to the area state directory, written by `lym area slack-setup` from the host. The outbox's reach becomes a D16 non-claim.
- F18: D13's Bound and D16's non-claim say that `lym area deploy` runs the landed commit's code, its deploy tooling included, on the host.
- F19 (fixed differently): D3 puts the supervisor in a transient scope `lym-area-<hub key>.scope` and has stop stop it and every recorded scope. New dead-supervisor criterion.
- F20 (fixed differently): D3's stop signals only pids in the area's scopes. Added to the stop criterion.
- F21: D2's start check covers every write grant, derived ones included. The criterion adds a hub root above `~/.config`.
- F22: D2's `lym area init` writes D6's lists and none of the study's `/tmp`, Go-cache or `~/.claude` rows. New criterion.
- F23: D15 renders other hubs' creel sockets as one path pattern, so the profile's text does not change when a hub records itself. D15's Bound states that the area-aware gate does not apply under AppArmor. The D15 criterion checks that the text is unchanged.
- F24: the second Open-risks line is replaced by a Constraint giving the measured `SCM_CREDENTIALS` row and `serveHubCall`'s behaviour.
- F25: the third Open-risks line is replaced by a Constraint from Neve's `/etc/nsswitch.conf` and `/etc/resolv.conf`. D14 step 1 reads the same on OSL-1046.
- F26 (fixed differently): D10 records the observed `.in_use/<pid>` writes, drops the host plugin-cache grant, and has the area install its own plugins. D11 cites gitcredentials(7).

## Self-check

- fix-claims, MEDIUM, `.in_use` files are not written only at session start: fixed. D10 now says they are written at session start and rewritten later.
- fix-claims, LOW, no step reads through the pidfd: fixed. D8 checks the pidfd's fdinfo `Pid:` after its `/proc` reads.
- fix-consistency, MEDIUM, the D8 criterion contradicted D3's inside test for the attach client: fixed. The criterion names another hub's daemon, and D8's rule now relies on the widened inside test (merged with fix-proposal 4).
- fix-consistency, LOW, the area state directory's list was incomplete: fixed. D3 lists the relay setup and the deploy clone (merged with fix-proposal 1).
- fix-consistency, LOW, the in-area `lym slack setup` had no stated outcome: fixed. It refuses, naming `lym area slack-setup`.
- fix-consistency, LOW, the creel socket was placed "under" the area root against "directly in": fixed. D6 places it directly in the area root.
- fix-way-forward, MEDIUM, a cold worktree could not be attached: fixed. `boot` takes a worktree under the hub and boots its session and watchdog from a restricted child. Attach asks it every time. The D3 Bound and the D6 criterion are updated.
- fix-way-forward, MEDIUM, the `lym machine start` notice pointed at a server that may not run: fixed. D7's way forward names `lym area start` first, and the operator pulls and starts from a server window.
- fix-way-forward, MEDIUM, local-directory marketplaces were unreadable inside: fixed. `lym area init` adds them to the `read` list, D10 says how updates reach the area, and there is a new criterion.
- fix-way-forward, LOW, the peer check would block `stop` when the supervisor is dead: fixed. `stop` sends the supervisor no verb.
- fix-proposal, MEDIUM, the F10 read grant exposed other hubs' deploy clones and relay setup: fixed. The grant is every hub's supervisor record alone.
- fix-proposal, MEDIUM, no watchdog spawn site was named: fixed. `boot` starts it under the area.
- fix-proposal, MEDIUM, D7's refusal contradicted cross-hub creel: fixed. Creel's cross-hub `deliver` and `confirm` are exempt.
- fix-proposal, MEDIUM, the any-seccomp-filter clause changed behaviour for hubs without an area: fixed. The clause is dropped in favour of recorded attach clients (merged with fix-consistency 1). F9 is now fixed differently.
- fix-proposal, LOW, there was no tmux-free way to detect a missing server: fixed. Attach asks `boot` every time, and `boot` does nothing when the session answers.

Every fork returned a report.

## Disputed

None.

## Deferred

None.

## Rulings

None.
