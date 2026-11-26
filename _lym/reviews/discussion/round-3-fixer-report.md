---
dispositions:
  - id: F33
    disposition: fixed
  - id: F34
    disposition: fixed
  - id: F35
    disposition: fixed
  - id: F36
    disposition: fixed
  - id: F37
    disposition: fixed
  - id: F38
    disposition: fixed
  - id: F39
    disposition: fixed
  - id: F40
    disposition: fixed
  - id: F41
    disposition: fixed
  - id: F42
    disposition: fixed
  - id: F43
    disposition: fixed
---

# Round 3 fixer report

Every premise was checked against the code before the fix:
`reedcli/window.go` and `desktophook.AfterWindow`, `attach_unix.go`'s `os.Environ()`, termwindow's `exec bash`,
`exitsweep.go` and `logger.NotifyExit`, `hubregistry.Lookup`'s Bolt read and `hubs.lock`, `fsx`'s `os.Rename`,
`slackengine/start.go`'s `O_APPEND|O_CREATE` log, the relay's `retire`, `SendReady`'s `RequireConfigured`,
the user-owned `app.slice/cgroup.procs` on Neve and the `directory` marketplace in `known_marketplaces.json`.
Every one held.

## Changes

All changes are to `_lym/discussion/decision-record.md` unless named otherwise.

- F33: D4's filter refuses `clone3` with `ENOSYS`, and callers fall back to `clone`.
  D3's inside test now rests on two bounds: Landlock denies the `cgroup.procs` write, and the filter refuses `clone3`.
  `lym area stop` cites those bounds.
  D15 installs the same filter rule, with no `connect` rule.
  A Constraints bullet records the `clone3`, Landlock, Neve, Go and glibc facts.
  New criteria: the D3, D4, D15 `clone3` escape fails under each barrier, and D4's criterion adds `clone3`.
- F34: D7 runs the area-file check wherever a hub is chosen: from the cwd, a path argument, a flag, the machine record or a shortname.
  A refusal ends the process with no later hub step, the exit log sweep included.
  New D7 criteria cover `lym reed window` with a `desktop.machine.yaml` script, and a refused verb that creates no file.
- F35: D6 adds the hub directory `<per-user lym state dir>/hubs/`, which holds only the machine record, its lock, the area registry and its lock.
  The token moves to its own directory, and the reads are granted on those directories.
  `~/.gitconfig` and `machine.json` stay file grants, renewed at the next `lym area stop` and `lym area start`.
  In-area readers take the lock read-only.
  Each daemon's machine-record entry now carries its shortname, socket path and repository name.
  A new D8 bullet has an area daemon take every other hub's data from host-written records alone.
  D8 moves the registry path, and D11 moves the token path.
  New criterion: a pane reads the machine record after a host daemon rewrites it.
- F36: D3 gives each hub one launcher (`lym area exec`) that lives as long as the supervisor and starts every area process as its own child.
  `boot` asks the launcher and never restricts a child.
  New criterion: a pane can signal a watchdog started by `boot`, and a surviving dobby after the server is booted again.
- F37: D6's attach client gets an allow-listed environment, and `boot` empties `update-environment`.
  `lym reed attach` waits until the supervisor has ended the attach scope.
  A window for an area hub starts no host shell; D13 says the same.
  New criterion covers the `detach-client -E` environment and the terminal.
- F38: D2 adds local marketplace paths and install locations to the host-side inputs, and refuses a hub that holds one, with its way forward.
  D2 and D10 limit the `read` entry to a marketplace outside the hub.
  A Constraints bullet records the Neve fact, and a new D2 criterion covers it.
- F39: a new D3 bullet: host-side processes open nothing under the hub or area root through a symlink, and write there only where a decision names the file.
  The forwarding file is read that way.
  D12 moves the relay's written files to `<area state dir>/relay/`.
  New D12 criterion: a planted link leaves the outside file unchanged.
  D12's Bound now says "creel mailbox" for what agents write; the outbox is the relay's own record (`internal/slackengine/outbound.go:1`).
- F40: D12 places each Slack record and names its writer and its reader.
  Presence and peers go through new `presence` and `status` requests, and `SendReady` reads no token.
  The relay takes its socket and shortname from the registry, and socket holders find an area hub's maps through the registry.
  D6 grants read on `<area state dir>/slack/`.
  New D12 criteria cover the cross-machine post, `lym slack away` and an operator message.
- F41: the `PATTERN-closed-area` bullet now reads "only its barrier decides a connect, from paths lym renders and the area file only `areaengine` reads".
  D15 states how the AppArmor barrier meets each clause.
  The gate's `pattern-change-over-cap` finding found the first wording ("the area's barrier") 5 characters over the 280 cap;
  "its barrier" brings the rule text to 278.
- F42: in Notes, `PATTERN-agent-addressing`'s Across hubs rule gains the area-registry repository name.
  The `PATTERN-child-processes` note names the Reap children paragraph's subreaper sentence and the supervisor's reaping.
- F43: D12's Bound and D16's non-claims add channel retirement through the beam records.
- `_lym/discussion/support-log.md`: a Round 3 entry records the facts read for this round.

## Self-check

Every finding below was fixed during the self-check.

- fix-claims, LOW, `update-environment` overclaim: fixed. D6 states that the allow list is the bound, and that an agent can reset the option.
- fix-claims, LOW, read-only lock open cannot create the lock: fixed. `lym area start` creates both lock files.
- fix-way-forward, MEDIUM, attach wait with a dead supervisor: fixed. The waiting attach stops the scope itself over the session bus.
  The D6 criterion now covers a killed supervisor.
- fix-way-forward, MEDIUM, machine-record move without migration: fixed. Readers also take the missing hubs' entries from the old `hubs.json`.
- fix-way-forward, LOW, the area window closes before the refusal is read: fixed. On a non-zero status it waits for a key.
- fix-way-forward, LOW, the launcher failure's way forward: fixed. The `boot` refusal names `lym area stop`, then `lym area start`, from a host shell.
- fix-consistency, LOW, D12's verb description is stale: fixed. The verb also serves `presence` and `status`.
- fix-consistency, LOW, D3's `boot` lacks `update-environment`: fixed in D3.
- fix-consistency, LOW, the D6, D8 criterion's socket source: fixed. Each side's source is now named.
- fix-consistency, LOW, the support log's count: fixed to ten BLOCKING.
- fix-proposal, MEDIUM, the in-area `ReadRelayState` reader: fixed. `status` carries the relay state, and in-area readers ask it.
- fix-proposal, MEDIUM, host connects follow planted links: fixed. The D3 rule adds connects through an `O_PATH` descriptor opened without symlinks.
- fix-proposal, LOW, the `~/.gitconfig` restart is untold: fixed. D16's README states it.

Every fork returned a report.

## Disputed

None.

## Deferred

None.

## Rulings

None reached this round.
