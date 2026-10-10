## Interview

The run is autonomous: no operator answered, and every question below was settled by the writer's own pick, recorded in the question ledger.

1. Board read: entry 246 with the seen-state (tier 3 CPU, 9,022 servers and 32,375 panes in 53 hours, four packages linking `lyx`, the #472 flake) and the goal list; related entries read for context: `bolt-and-store` (hub store gc, running), `daemon-wakeups` (watchdog polls, running), `proc-child-lifetime` (note), `gate-test-scope` and `shuttle-send` (done).
2. Codebase read at coarse level: `cmd/testtiming` runs `go test -json ./...` and has a redundancy mode; `gitkit` already builds a repo template once per binary and copies it; `hubforge` copies bares once per binary and clones and wires every hub fresh; `tmuxkit` isolates `TMUX_TMPDIR` and kills servers at cleanup; `lyxbin` builds `./cmd/lyx` per call, with `loomcli` keeping its own once-per-package wrapper; the hermetic git config already sets `gc.auto=0` and `maintenance.auto=false`; reed starts its server with its own shell as the first pane's command and splits strand panes with no command, so they are login shells; `lyx gate test` holds a slot and runs `go test` in a process group; reed's clean environment strips only Claude and agent keys, so the hermetic git variables pass through; the watchdog is a detached daemon that idles out on its own schedule, which existing tmux tests tolerate.
3. Measurement: how to get CPU, peak memory and process count per package.
   Picked cgroup counters through a transient user scope, rusage as the fallback, the system-wide fork counter for process count, and trace2 per-process files for the git census.
4. Fixture versus code split: picked a marker variable on fixture git spawns captured by trace2's env-var logging.
5. Non-login shells: picked a tmuxkit-owned pre-started server with `default-command`, no production change.
6. Shared servers: picked a package server for tests that name their own sessions; hub-driven tests keep the hub key.
7. No tmux in tiers 1 and 2: picked a runtime socket check in `tmuxkit.Main` keyed on build tags, plus tightening the static scan.
8. `lyx` once: picked an environment variable set by `lyx gate test` and the harness, read by `lyxbin`, denied to agents.
9. Lingering children: picked a private `TMPDIR` per test binary with a `/proc` scan after `m.Run`, and a group-killing runner in `lyxbin`.
10. Fixture reuse: picked a relocating hub copy from a per-shape template plus a shared read-only template with a fingerprint check.
11. Fewer spawns in `lyx`: picked in-process geometry reads in `lyxcwd` and `gitrepo`, go-git ref reads for `fabricengine`'s `rev-parse` sites, parity rows for each.
12. `PATTERN-shuttle-stop`: picked a call-site scan with the entry's exemption list.

## Rejected alternatives

- Per-package resource columns from `go test -json` alone: it carries wall time only.
- Counting processes with `strace -f`: perturbs timing and needs a second run; the fork counter is free and the report states its noise.
- Counting processes with `perf` tracepoints or process accounting: need privileges the harness cannot assume.
- Peak memory from rusage only: `ru_maxrss` is the largest single descendant, not the tree; kept as the fallback, marked.
- Running the harness unslotted: a module-wide test run outside the hub's bound is what `PATTERN-gate-slot` exists to stop.
- A non-login shell by changing reed's split call or `default-shell`: changes production pane start mode, which an earlier shared decision pinned.
- A non-login shell by pointing `$SHELL` at a wrapper script: fragile, and leaves the operator's `~/.tmux.conf` in every test server.
- Sharing the hub-keyed reed server across hubs: needs a test-only override of `reedengine.ServerName`, a production seam for a test concern.
- Enforcing tmux-free tiers by static scan alone: starting tmux has no static shape, so a `lyx` child starting reed would pass.
- Building `lyx` once into a content-addressed cache under the Go build cache: a stale binary from another tree is a silent wrong answer; a per-run build named by the gate is always fresh.
- `lyxbin` building once per test binary into an unremovable temp directory: the private `TMPDIR` of D8 gives it a lifetime.
- Four per-package `lyx` runners each adding group kill: `PATTERN-testkit` forbids duplicated test support.
- Only `gc.autoDetach=false` for #472: the hermetic config already disables auto gc, so the flake's writer is a lingering `lyx` child, which the leftover check names.
- Byte-copying a hub: links and worktree pointers carry absolute paths, as `hubforge`'s doc already records.
- A hub copy that rewrites only known path sites: every missed site is a silent wrong fixture; the generic text rewrite plus a verification scan needs no site list.
- Fingerprinting the shared hub with `git status`: one git spawn per test, which the task is removing; the file-tree fingerprint spawns nothing.
- A package-end fingerprint in `tmuxkit.Main`: kits may not import `hubforge`.
- Caching HEAD or refs for the process's life: they change under a process; only geometry is immutable while a worktree exists.
- Reading geometry through go-git's dotgit detection: go-git does not expose the common dir cleanly for a linked worktree, and the `.git` and `commondir` files are two reads.
- Dropping the `rev-parse --disambiguate` and `<commit>:<path>` spawns: their answers are git's own disambiguation, which no in-process read reproduces.
- Pinning `PATTERN-shuttle-stop` by review alone: it was already found false by review with nothing to catch it.

## Review rounds

_No rounds yet._

## Question ledger

- Auto-pick: measurement mechanism.
  Picked cgroup counters via `systemd-run --user --scope` with rusage fallback, `/proc/stat` forks, trace2 files.
  Rationale: the only privilege-free sources that sum a process tree, and this machine has them (`systemd-run`, cgroup v2 `memory.peak` and `pids.peak` present).
- Auto-pick: serial per-package runs under `-resources`, with the default mode unchanged.
  Rationale: per-package counters need one child per package; the default stays the cheap whole-module run.
- Auto-pick: the harness takes a gate slot.
  Rationale: `PATTERN-gate-slot`.
- Auto-pick: fixture marker through `GIT_TRACE2_ENV_VARS`.
  Rationale: git 2.53 here supports `trace2.envVars`; no production code changes.
- Auto-pick: before-state measured at the first card in the pair, load average recorded.
  Rationale: an agent cannot make the machine quiet; the report says what the load was and names the regenerating command.
- Auto-pick: `default-command` is `$SHELL` non-login, rc files as the shell's default.
  Rationale: reed's prelude and launch script expect the operator's shell; the login profile is what the census blamed.
- Auto-pick: `exit-empty off` on pre-started servers.
  Rationale: a session-less server exits otherwise; tmux 3.6 here documents the option.
- Auto-pick: package server only for tests naming their own sessions.
  Rationale: the hub key is production naming.
- Auto-pick: tier-2 reed-starting tests split into a `tmux` file rather than tagged whole.
  Rationale: keeps the reconcile assertions in the batch gate.
- Auto-pick: the prebuilt-binary variable is denied to agents.
  Rationale: a forged value is a silent wrong binary for every package.
- Auto-pick: private `TMPDIR` set in `tmuxkit.Main`.
  Rationale: the one entry point every tagged package already calls.
- Auto-pick: watchdog leftovers killed without failing.
  Rationale: `daemon-wakeups` owns the watchdog's exit schedule; failing here would duplicate that task.
- Auto-pick: `lyxbin` gains the runner and `internal/proc` import.
  Rationale: `PATTERN-testkit` forbids four copies, and `proc` is not on the kit import ban list; the bound is widened in the entry.
- Auto-pick: hub shapes are a closed set in `hubforge`, starting with two anchors times zero or one pair.
  Rationale: a template per arbitrary shape would rebuild per test again.
- Auto-pick: shared-hub write check per sharing test, by cleanup fingerprint.
  Rationale: no package-end hook can live in a kit, and a per-test check blames closer than a package-end one.
- Auto-pick: relocation by generic text rewrite plus verification scan.
  Rationale: no site list to go stale.
- Auto-pick: only geometry is cached in `gitrepo`.
  Rationale: refs change under a process.
- Auto-pick: `GIT_DIR` and `GIT_WORK_TREE` not honoured by in-process reads.
  Rationale: lyx never documented them, and honouring them would need a spawn to be faithful.
- Auto-pick: `RemoveStrand` scan allowlist includes `reedcli` and `shuttlefake`.
  Rationale: reed's own verb and the fake are not shuttle-run stops.
- Resolved from the codebase: the hermetic config already sets `gc.auto=0` and `maintenance.auto=false`, so #472's fix is the leftover check plus `autoDetach=false`.
- Resolved from the codebase: reed's clean environment strips only Claude and agent keys, so the hermetic git variables and `TMUX_TMPDIR` reach test servers.
- Environment: nothing outside the two discussion files was found broken; `lyx loom validate-discussion` ran before writing and reported only the missing files.
