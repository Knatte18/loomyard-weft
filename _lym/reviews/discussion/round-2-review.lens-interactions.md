```yaml
lens: interactions
outcome: reported
```

Task: interactions lens. Below are the pairs of decisions I checked, then the findings.

**Pairs checked**
- D3 inside test × D6 attach-client record × D8 cross-hub refusal: **broken** (F-I1)
- D4 supervisor death × D8 refusal: **broken** (F-I1)
- D6 grant on "every hub's supervisor record" × D8 inside test for an area hub started later: **broken** (F-I1)
- D7 cwd refusal × D13 `window` and D3 `boot` "a worktree under the hub" × D6 host-side attach: **broken** (F-I2)
- D7 "cross-hub creel reads … never its hub config" × D8 receive path (`Lookup`, `RepositoryName`, `trustedSender`): **broken** (F-I3)
- D2 start-check fence × D6 read/exec grants × D10 marketplace `read` entries: **broken** (F-I4)
- D15 profile rendered from "the Landlock paths" × D6 per-hub supervisor-record grants: **broken** (F-I5)
- D5 and D6 machine-record fields (socket path, area-aware level) × D8 older-binary daemons rewriting the record: **broken** (F-I6)
- D3 stop by scopes × D9 recorded pane and run scopes × D9 `kill-scope`: held
- D3 stop × D13 `window` Konsole and D12 relay (the supervisor's host-side children in its scope): held; the record says stop ends them
- D9 bound (barrier plus descent) × D3 inside test (attach clients counted): held; the two are consistent, and the D9 criterion matches
- D3 subreaper × PATTERN-child-processes double-fork reap (declared change "only an area supervisor subreaps"): held
- D3 `-S` argv × D6 `TMUX_TMPDIR` at the area root × reed's `-L <key>` inside: held, provided `-S` equals `<area root>/tmux-<uid>/<key>`
- D3 server env × D10 `CLAUDE_CONFIG_DIR` and `CLAUDE_CODE_TMPDIR` on the launch line × D11 token removal: held
- D11 token file × D2 fence × D6 read grant: held
- D12 setup in the area state directory × D3 "state dir holds nothing the area writes": held
- D3 spawn-site refusals × D8 "a hub without an area never starts an area hub's daemon" × D7 creel exemptions: held
- D4 supervisor death × D9 cap fallback (`RLIMIT_DATA`) × D12 relay in the supervisor's scope: held
- D14 stage order × D9 and D12 verbs in stage 1, D13 in stage 2: held
- Acceptance criteria against the decisions they touch: D2/D5 fence, D3 stop and restart, D6/D8 cross-hub, D9 refusals all held. The D3/D15 inside-test criterion is satisfiable but does not catch F-I1, since it tests only a pane, a host process and the supervisor's children.

**Findings**

- **F-I1** — BLOCKING, design, `_lym/discussion/decision-record.md:90` (D3 inside test), with D6:144 and D8:201–203.
  - Summary: D3's inside test fails open, so D8's "refuse every verb but deliver and confirm" misses filter-carrying processes whose ancestry no longer reaches a known supervisor. Those processes resolve as `EXTERNAL` with every verb at another hub's daemon. Three concrete paths:
    1. The attach client's `detach-client -E` command connects to another hub's creel socket. Its connects are decided under the client's `PR_SET_PTRACER`. It forks a child that inherits the connected fd, then exits. The child is reparented to `systemd --user` (the client is no descendant of the subreaper supervisor), so its ancestry reaches neither the supervisor nor a listed attach client. It still carries the filter, and it sends on the inherited socket.
    2. D4 supervisor death: survivors holding already-connected creel sockets are reparented away from the dead supervisor, and their posts pass as outside.
    3. D6 grants read on "every hub's supervisor record" as a fixed Landlock ruleset at restrict time. Hub A's in-area daemon cannot read the record of an area hub B started after A, so every peer from B fails the test.
  - Evidence: decision-record.md:90–92 (test needs barrier **and** ancestry); :172 (the client's forks are no supervisor descendants); :121 (a dead supervisor); :67 and :144 ("alone" and per-record grants); probe `superviseNotifications` (subreaper reparenting, study table row "connect from an orphan").
  - Fix: make the test fail closed for cross-hub verbs. A peer carrying the hub barrier (any seccomp filter, or an area profile label) whose ancestry reaches no readable supervisor record is refused every verb but `deliver` and `confirm`. Or grant read on the parent directory of all supervisor records. Add an acceptance row for an orphan of the attach client's command posting on an inherited socket.

- **F-I2** — BLOCKING, design, `_lym/discussion/decision-record.md:180` (D7), with D13:278 and D3:77.
  - Summary: `window` and `boot` admit "a worktree under the hub, checked by canonical path". Host-side `lym reed attach` derives its hub as the parent of the nearest `.git` root (`lymcwd.gitWorktreeRoot`, then `filepath.Dir`). So an agent can do this:
    - create `<hub>/x/wt/.git`, plus `<hub>/x/_bolt/_lym` and `<hub>/x/.../reed.yaml` with `tmux: <hub>/x/evil`;
    - ask `window` for `<hub>/x/wt`.
    The host attach then resolves hub `<hub>/x`, whose `ServerName` has no area file, so D7 does not refuse it. The non-area path runs: the root pre-run's stencil seed runs git over the fake Bolt, `wireLocation` loads the fake `reed.yaml`, and `CheckTmuxFloor(cfg.Tmux)` runs the agent's binary on the host.
  - Evidence: `internal/lymcwd/lymcwd.go:121–126` and `:202–211`; `internal/reedcli/cli.go:107–115`; `internal/reedcli/attach.go:55`; `cmd/lym/stencilseed.go:52–59`; `internal/reedengine/server.go:47–52`; termwindow `konsoleArguments` (`--workdir dir`).
  - Fix: D7's check refuses when the canonical cwd lies anywhere under an area hub's root, not only when the resolved hub is an area hub. D13 and D3 admit only a direct child of the hub root. Add an acceptance row for a nested fake hub.

- **F-I3** — BLOCKING, design, `_lym/discussion/decision-record.md:188` (D7), with D8:201–212.
  - Summary: D7 says cross-hub creel reads an area hub's machine-record entry alone, never its hub config. But a host hub's daemon receiving `deliver` from an area hub still does three reads of area-written data:
    - `hubs.Lookup`, which reads the area hub's Bolt shortname file;
    - `CreelGeometry`, which reads the same file;
    - `RepositoryName` → `HubOriginURL`, which reads the area prime's `.git/config` origin through go-git.
    `trustedSender` grants trusted-orch authority by that repository name. Area agents can rewrite their origin URL to a repository the receiving hub's `creel.yaml` trusts, and their orch's mail is then stamped trusted as another repository's orch. That is authority outside the area, beyond D16's "trust in an area hub" non-claim, since the area hub poses as a different hub.
  - Evidence: `internal/creelengine/receive.go:133`, `:170–178`, `:294–305`; `internal/creelcli/hubdirectory.go:62–71`, `:84–86`; `internal/hubregistry/repository.go:37–49`; `internal/fabricengine/codebinding.go:71–73`; `internal/gitrepo/remote.go:47–57`; `internal/hubgeom/creelgeom.go:19–31`.
  - Fix: in D8, take an area hub's shortname and repository name from its machine-record entry, written by host-side `lym area start` from host-read data at start, or state that an area hub's mail is never stamped trusted. Make D7's sentence match.

- **F-I4** — MEDIUM, design, `_lym/discussion/decision-record.md:55` (D2 start check), with D6:142–144 and D10:243.
  - Summary: D2's fence omits host-side inputs that other decisions name. An operator `write` entry on any of them passes the start check:
    - `~/.gitconfig` and `~/.config/git`, which host git runs (for example `core.fsmonitor` or `core.hooksPath`);
    - `~/.config/lym/machine.json` and the machine hub record, which D5 and D8 rely on being read-only inside;
    - the Claude Code launcher and versions, which the host executes;
    - the local marketplace directories, which host Claude Code installs plugins from.
    Each breaks the Goal's no-persistence claim or D5's bound.
  - Evidence: decision-record.md:55–58 against :142–144, :205–207 and :243.
  - Fix: derive D2's fence from every D6 read/exec grant plus the D10 marketplace entries, rather than a separate list.

- **F-I5** — MEDIUM, consistency, `_lym/discussion/decision-record.md:304` (D15), with D6:144.
  - Summary: D15 renders the profile from "the area file's lists and the Landlock paths" and claims its text does not change for new hubs. But D6's Landlock paths include a read grant on every hub's supervisor record, so each new area hub changes every AppArmor hub's rendered text and hash. The start then refuses until a root reload, contradicting "a new hub is reachable from an AppArmor area with no reload".
  - Evidence: decision-record.md:144, :304–310.
  - Fix: render the supervisor records as one path pattern, as D15 already does for creel sockets.

- **F-I6** — MEDIUM, design, `_lym/discussion/decision-record.md:157` (D6), with D5:127 and D8:205–207.
  - Summary: the record adds per-entry fields to the machine record: the socket path and the area-aware level. But every daemon's `hubregistry.Record` rewrites the whole file through a typed `recordFile` round trip, which drops keys the binary does not know. A host hub still on an older binary therefore strips the level and socket path from every entry each time it records itself. The area hubs that `lym area start` wrote then become unreachable for cross-hub creel until the next `lym area start`. D5 only considers older daemons as unreachable targets, not as writers that erase the fields.
  - Evidence: `internal/hubregistry/hubregistry.go:37–49`, `:55–66`; `internal/state/state.go:157–178`.
  - Fix: state that the new fields survive an older writer (keep unknown keys on rewrite, or keep them in a separate file only area-aware binaries write), or name the outage and its way forward.

**Tool calls**
- `internal/state/state.go` `UpdateJSON` and `internal/gitrepo/remote.go` `RemoteURL`: my context lacked whether a record rewrite keeps unknown keys, and whether reading the origin URL spawns git.
