```yaml
lens: code-pattern-fit
outcome: reported
```

Task: code-pattern-fit lens, the diff checked against `PATTERN.md`/`pattern/`, `CLAUDE.md`, declared PATTERN changes and commit trailers.

## Rules checked

| Rule | Verdict | Evidence |
|---|---|---|
| `PATTERN-leaf-packages` (`selfexec` closed import set) | Held | `selfexec.go` `CheckWith` adds only `os` and `logger` calls and takes callbacks; no new import. |
| `PATTERN-cwd-resolution` | Held | `dobbyengine` (`loop.go`, `start.go`, `logfile.go`) imports no `lymcwd`. `OutPath` and `PrimeLogPath` arrive through the told `Geometry`. `hubgeom.DobbyBeamDir` is the one shared path rule. |
| `PATTERN-durable-vs-ephemeral-state` | Held | `dobby.out` is named beside `dobby.log` in `dobbyengine`, with its path in `hubgeom`. `check.json`, `check.out` and `check.lock` are named only in `internal/adoptcli`, under `fabricengine.AdoptScratchDir`. |
| `PATTERN-tolerant-records` | Held | `State.StartTicks` and `LandingsMachineOnly`, `HistoryEntry.Machine` and `CheckRecord` are all `omitempty` or tolerant. Writes go through `state.WriteJSON`. `ReadCheck` reads a record that fails to decode as absent. |
| `PATTERN-daemon-wakeups` | Held | The heartbeat runs inside `loop`'s existing tick (`now.Sub(d.lastHeartbeat) >= heartbeatInterval`). No new timer or wake. |
| `PATTERN-spawn-observability` | Held | `spawnDetached` and `startSuite` both log the spawn through `logger.Info` or `logger.Warn`. |
| `PATTERN-run-process-owner` | Held | `registerDobby` now runs only inside the `Deps.Locked` seam, after the lock is held. |
| `PATTERN-gate-slot` | Held | `gatecli.RunGate` goes through the same `runTest`. The observer hooks sit in `runWithSlots`, `report` and `reportBlocked`. |
| `PATTERN-config-layers` and `PATTERN-sparse-hub-base` | Held | `adopt` is seeded only through `Module.MigrateHubBase` in `ReconcileHubWideAt`: the present-file branch, and the absent-file branch for prime and legacy seeds. `MigrateConfig` no longer seeds, so the local layer, `sparseSeed` and `run_settings` stay unseeded. |
| `PATTERN-pane-text-injection` | Held | Every notice goes through `orchcli.NotifyFromScoped` or `NotifyFrom`. |
| `PATTERN-refusal-way-forward` | Held | The batten refusal on a `machinename.Name` error (`arm.go`) names `give the machine a hostname, then re-run lym batten run <slug>`. The dobby refusal and notice lines name `lym beam start`. |
| `PATTERN-test-speed` | Held | The `adopt` row is added to the gate-key table in `pattern/PATTERN-test-speed.md`. The binary-replacing test is in `dobbyreexec_integration_test.go`; I did not verify its `tierN` tag. |
| `PATTERN-cli-cobra` | Held | `check` is a hidden subcommand under the `adopt` root. |
| `PATTERN-gitrepo-client-boundary` | Held, see F-a | `worktreeGone` (`loop.go`) is an `os.Stat` of `<WorktreeRoot>/.git` with a hand-written literal. It runs no git. |
| `CLAUDE.md` docs in the same commit | Held | `doc.go` changed for `dobbyengine`, `beamcli`, `adoptcli`, `gateslot`, `gatecli`, `shedengine` and `selfexec`. `docs/overview.md` changed for both the adopt check and the dobby log. `pattern/PATTERN-test-speed.md` changed. The cards and the sampled files show no stale `--gate finalize` for the check. |
| `CLAUDE.md` semantic line breaks | Held for `docs/overview.md` | Each sentence is on its own line. I did not read `docs/dotnet-hub.md` (one line changed). |
| Declared PATTERN changes landed | Held | The directive in `instruction-1` and the decision record declare no PATTERN add, change or remove. The `PATTERN.md` and `pattern/` diff is the one table row. |
| Commit messages and `Pattern-Departure:` trailers | Held | No commit carries a trailer, and none is needed: no difference from the answer key cites a PATTERN entry. |
| Comment conventions | Held | No end-of-line comments and no `/* */` blocks in the new code. Line wrapping is the lint gate's. |

## Findings

**F-a. NIT, class `consistency`.**
- Location: `internal/dobbyengine/loop.go` `worktreeGone`, the `filepath.Join(d.geom.WorktreeRoot, ".git")` line.
- The worktree's git-link name is a bare `".git"` literal in `dobbyengine`. `PATTERN-gitrepo-client-boundary` keeps local git reads in `internal/gitrepo` (`dotgit`). Decision 1 does prescribe "the worktree's git link joined with `.git`", and this is an existence stat that runs no git, so it is at most a NIT.
- Fix: leave it, or have the told `Geometry` carry the git-link path so `dobbyengine` names no git file.
- Evidence: diff of `loop.go`, `worktreeGone`.

**F-b. NIT, class `consistency`.**
- Location: `internal/dobbyengine/loop.go`, the `Deps.Log` doc and the `Deps.CheckBinary` doc.
- The `Deps.Log` doc says "the dobby passes Geometry.LogPath". That ties the comment to another named symbol, which `code-quality`'s Self-contained rule forbids; the other new doc comments I read avoid it.
- Fix: reword the `Deps.Log` doc to say rotate is set only while the lock is held, without naming `Geometry.LogPath`.
- Evidence: the `Deps.Log` comment in the `loop.go` diff.

## Outside my lens (noted only)
`internal/beamcli/dobbyreopen_unix.go` is built for every non-Windows platform and calls `unix.Dup2`. `golang.org/x/sys/unix` has no `Dup2` on linux/arm64, so that target would fail to build. `Dup3` or a build-tagged split would avoid it. This is correctness, not a pattern rule, so the handler or the code-correctness lens should judge it.

## Tool calls (4 commands in one Bash call)
- `git diff --stat ... -- PATTERN.md pattern CLAUDE.md contracts`: to see which PATTERN files the diff changes. My context only had a truncated stat.
- `git log ... | grep -ci Pattern-Departure`: to count the trailers. I had read the subjects but not the trailers.
- `grep -rn 'DotGitName|".git"' internal`: to see how other packages name the git link. The result confirms bare `".git"` literals are common, so F-a stays a NIT.
- `grep -rn 'dobby.out|OutFileName' internal`: to see where `dobby.out` is named. The names sit beside their `dobby.log` siblings.
