```yaml
lens: existing-paths
outcome: reported
```

Task: existing-paths lens. I traced the existing code paths that write or read each state the record's decisions introduce or change.

**Paths traced, per decision:**

- **D2, the area file and the hub key** (`reedengine.ServerName(hubPath)`, server.go:47). The hub key is derived from the path alone. **Survives** for a real hub. It **breaks** when the cwd resolves to a nested repository, because `lymcwd.resolveCore` takes `hubPath = filepath.Dir(workTreeRoot)` (lymcwd.go:121-132). See finding 1.
- **D3, the server spawn** (`serverSpawnArgv` with `-L`, `-f /dev/null` and `cfg.Shell`, lifecycle.go:622-644; `stripTraceID` and `stripIdentityEnv`, lifecycle.go:675-702). In-area reed uses `-L` under `TMUX_TMPDIR`, so the socket lands at the path the supervisor's `-S` derives. **Survives.**
- **D3, `lym area stop` over runproc records** (`ender.end` and `signalGroup`, end.go:192-229). The cgroup filter D3 adds bounds the pids. **Survives.**
- **D5 and D8, the machine record** (`hubregistry.Record` rewrites the whole entry list, hubregistry.go:55-67). Other hubs' entries are copied whole, so any fields an entry gains survive a rewrite by a current binary. Whether a rewrite by an older binary keeps them is unverified. **Survives**, but the record is silent on it, and the incremental rule bars a finding below BLOCKING.
- **D6 and D7, host-side `lym reed attach`.**
  - The reed group pre-run's `wireLocation` (cli.go:104-120) calls `LoadConfig` and `RevivalFilter`, and D6 states that attach reads no hub config before it restricts itself. **Survives** for a cwd in a real worktree.
  - It **breaks** for a nested repository reached through D13's `window`. See finding 1.
- **D7, root pre-run steps** (`seedStencils` → `stencilSeedTarget` → `preflight.HubPresent`, then a Bolt pull and commit, stencilseed.go:52-59 and 81-97). Exempt verbs skip these for an area hub. They **break** for the nested repository (finding 1).
- **D7 and D8, the receiving daemon's lookups of a sending area hub** (receive.go:133, 170-178, 282-304; hubdirectory.go:62-95; creelgeom.go:18-35; hubregistry.go:72-95; repository.go:37-50; gitrepo/remote.go:47-59).
  - The daemon reads the area hub's Bolt shortname file (`Lookup`, `CreelGeometry`) and its prime's `.git/config` origin URL (`RepositoryName`, through go-git in-process).
  - It decides the trusted stamp from that URL.
  - **Breaks** D7's "Cross-hub creel reads an area hub's machine-record entry alone". See finding 2.
- **D6 and D8, reading another hub's supervisor record from inside an area.** The Landlock ruleset is fixed at `restrict_self`, and D6 grants read on "every hub's supervisor record alone", that is, per file. A record created after this area started is unreadable inside. **Breaks** D8 for hubs started later. See finding 3.
- **D9, the pane cap** (`capPaneMemoryLocked` → `proc.CapProcessMemory`, panememory.go:33-53) and the run cap (`runMemoryCapped`, `scopeCgroup`, `killScope`, memorycap_linux.go:146-262). D9 replaces the bus call, the marker and the `cgroup.kill` write with verbs. **Survives.**
- **D12, the relay spawn** (`spawnDetached` with `cmd.Dir = geom.HubPath`, start.go:83-109; `StopRelay` → `runproc.EndRole`, start.go:146-148). D12 replaces both. The relay verb carries the skip-stencil annotation (slackcli/cli.go:163). **Survives.**
- **D13, `window`** (`konsoleArguments`, termwindow.go:95-96). The worktree the area hands over becomes Konsole's `--workdir`, and the host-side attach resolves its hub from that directory. **Breaks**: finding 1.

**Findings**

1. **BLOCKING, design.** Location: `_lym/discussion/decision-record.md:278` (D13 `window`) and `:180-183` (D7 cwd check).
   - Summary: D13's `window` and D3's `boot` accept any worktree "under the hub". Host lym resolves a hub as the parent directory of the nearest `.git` (`lymcwd.resolveCore`).
   - So an agent can build a nested repository such as `<hub>/x/wt`, with a fake `<hub>/x/_bolt/_lym` and `_lym/config/reed.yaml`. Its cwd resolves to the hub `<hub>/x`, which has no area file.
   - D7's check therefore passes. The Konsole-launched host `lym reed attach` then loads the fake hub's reed config and runs its `tmux` (`CheckTmuxFloor` on `cfg.Tmux`). The root pre-run's stencil seed also pulls and commits in an agent-written `.git` on the host.
   - Evidence: lymcwd.go:121-132; cli.go:104-115; attach.go:55; stencilseed.go:52-59; termwindow.go:95-96.
   - Fix: D7 refuses when the canonical cwd lies anywhere under an area hub's root. `window` and `boot` take only a direct child of the hub root.

2. **BLOCKING, design.** Location: `_lym/discussion/decision-record.md:188` (D7, cross-hub creel line), with D8 at `:201-212`.
   - Summary: a receiving daemon looks up the sender's hub by the Bolt shortname file (`hubregistry.Lookup`) and its socket through `CreelGeometry`, both read from the area hub. It also stamps trust from `RepositoryName`, which reads the area hub's prime `.git/config` origin URL; agents write both.
   - An area agent can set `remote.origin.url` to a repository another hub trusts and keep pushing through `pushurl`. Its orch's mail is then stamped `trusted` as that other repository's orch.
   - That is more than D8 and D16's stated bound, which is trust in the area hub itself. It also contradicts D7's "machine-record entry alone".
   - Evidence: receive.go:133, 170-178, 282-304; hubdirectory.go:62-86; repository.go:37-50; gitrepo/remote.go:47-59; creelgeom.go:18-23.
   - Fix: for an area hub, the record states the repository name and shortname that `lym area start` writes into its machine-record entry from host-side input. The receiving daemon reads those, never the hub's Bolt or `.git/config`.

3. **BLOCKING, design.** Location: `_lym/discussion/decision-record.md:144` (D6 read grant), with D8 at `:204`.
   - Summary: D6 grants read on "every hub's supervisor record alone" inside a ruleset fixed at `landlock_restrict_self`. An area hub started after this area cannot have its record read by this area's daemon, so that daemon cannot run D3's inside test for the later hub's agents.
   - D8 does not say how a daemon treats such a peer. As written, the peer fails the test and gets every verb, so it can post as `EXTERNAL`, the act D8 exists to refuse.
   - Evidence: the record's D3 inside test (line 90-92) and D8 (line 201-204). The Landlock ruleset is fixed once applied (study, Landlock section).
   - Fix: grant read on the area state directory's parent, or carry each supervisor pid in the machine-record entry, which D6 already grants read on. D8 refuses every verb but `deliver` and `confirm` when a filtered peer's area cannot be read.

Tool calls:
1. `grep` of `gitrepo.RemoteURL`, the `RepositoryName` and `trustedSender` callers, and `pushurl`. My context did not show whether the origin read runs git or how its result reaches the trust stamp.
