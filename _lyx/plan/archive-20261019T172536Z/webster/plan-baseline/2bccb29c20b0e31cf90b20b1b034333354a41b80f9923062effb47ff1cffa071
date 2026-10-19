# Card 4 — gitexec-floor-check

**Edit:**
- `internal/gitexec#runCore`
- `internal/gitexec#Run`
- `internal/gitexec#RunStdin`
- `internal/gitexec#RunGit`
- `internal/gitexec#SetKillReporter`
- `internal/gitexec/doc.go`
- `cmd/lyx#spawnObservabilityAllowedSpawners`
- `cmd/lyx#logGitKill`

**Create:**
- `plan:internal/gitexec#CheckFloor` -> `func CheckFloor() error`
- `plan:internal/gitexec#floorCache` -> `type floorCache struct`
- `plan:internal/gitexec#floorCache.check` -> `func (c *floorCache) check(probe func() (string, error)) error`
- `plan:internal/gitexec#gitFloor` -> `var gitFloor floorCache`
- `plan:internal/gitexec#probeGitVersion` -> `func probeGitVersion() (string, error)`
- `plan:internal/gitexec#versionProbe` -> `var versionProbe = probeGitVersion`
- `plan:internal/gitexec#SetVersionProbeForTest` -> `func SetVersionProbeForTest(t *testing.T, probe func() (string, error))`
- `plan:internal/gitexec_test#TestCheckFloor` -> `func TestCheckFloor(t *testing.T)`

**Uses:**
- `plan:internal/toolfloor#CheckGit`
- `plan:internal/toolfloor#FloorError`

**Intent:** D4, git half: a process whose git is below the floor fails every git command with the floor error, checked once per process before the first git command.

- `probeGitVersion` runs `git --version` with `exec.Command` directly, never through `runCore`, with the same `childEnv(nil)` and `proc.HideWindow`, and logs the spawn with `logger.Debug`; it returns the trimmed stdout, or the exec error when git could not run.
  It bypasses `runCore` only because `runCore`'s first step is the floor check itself, so a probe through it would re-enter the check it serves while `check` holds the cache's mutex; it keeps `runCore`'s local-command environment, so the probe runs as any local git command does.
  `versionProbe` holds it so a test injects a version line; tests never substitute a git binary.
- `floorCache` holds a mutex, a done flag and the cached floor error.
  `check` returns the cached result once done; otherwise it calls probe: an exec error is returned and not cached, so the next command probes again; a line `toolfloor.CheckGit` parses is cached with its answer (nil or the `*toolfloor.FloorError`); an unparseable line is logged once with `logger.Warn` naming the line and cached as admitted.
  Bound: only a parsed version below the floor is refused, nothing bypasses a parsed refusal, and a long-lived daemon keeps its cached answer after a git upgrade during its life.
- `CheckFloor` returns `gitFloor.check(versionProbe)`, exported so a git spawn outside the exec core (card 5's `planglyph.Delta`) checks too.
- `runCore` calls `CheckFloor` first and returns its error unwrapped with exit code -1, before any spawn, so `Run`, `RunStdin` and `RunGit` all return the floor error, which is no `*GitError`.
  `runCore`'s godoc, the godocs of `Run`, `RunStdin` and `RunGit`, `gitexec.go`'s file header comment and the package summary in `internal/gitexec/doc.go` name the floor error as the other non-`*GitError` result, which no git command ran to produce, beside the exec-level failure where git could not be run at all.
  It also logs each spawn it makes with `logger.Debug` (subcommand and dir), so the exec core is observable; `internal/logger`'s import closure no longer reaches `internal/gitexec`, so the import closes no cycle.
- `cmd/lyx#spawnObservabilityAllowedSpawners` drops its `internal/gitexec/gitexec.go` entry, whose import-cycle reason no longer holds; `TestSpawnObservability_ProductionSpawnsAreLogged` then holds gitexec to the rule like any file.
- `SetKillReporter`'s godoc and `cmd/lyx`'s `logGitKill` godoc stop saying gitexec cannot log: the reporter stays, since the process owner decides how a kill is reported.
- `internal/gitexec/doc.go` gains a `# Floor` section: the once-per-process check, the uncached exec failure, the Warn-and-admit of an unparseable line, the exported check and the bound above.
- `SetVersionProbeForTest`, in `export_test.go`, replaces `versionProbe` and resets `gitFloor` for the test, restoring both on cleanup; its godoc says it changes process-global state, so the caller must not be parallel.
- `TestCheckFloor` goes in the `//go:build integration` file `gitexec_test.go`, since it spawns git through `Run` and `cmd/lyx`'s tier-purity guard bans `gitexec.Run` in untagged test files; it is a serial table test (no `t.Parallel`, naming the process-global probe and cache), each row installing a counting probe through `SetVersionProbeForTest`:
  "a below-floor git refuses every entry point" injects `git version 2.42.0` and asserts that `Run`, `RunGit` and `CheckFloor` each return a `*toolfloor.FloorError` that `errors.As` does not read as `*GitError`, and that the probe ran once across the three calls;
  "an at-floor git is admitted once" injects `git version 2.43.0`, runs `Run([]string{"--version"}, dir)` twice and asserts success and one probe;
  "a probe that could not run is retried" scripts an exec error then `git version 2.43.0`, asserts the first `CheckFloor` returns that error, the second nil, and two probes;
  "an unparseable line is admitted with a warning" injects `git version unknown`, asserts `Run` succeeds and that `logcapture` holds one Warn naming the line, then runs `Run` again and asserts it succeeds with still one probe and one Warn, so the admitted answer is cached.
  Go starts the package's parallel tests only after its serial ones finish, so no parallel test sees an injected line.

**ImpactSummary:** Every git command lyx runs through gitexec gains a once-per-process version check and a Debug spawn line; callers see a new non-`GitError` refusal only on a git below 2.43, and one allowlist entry leaves the spawn-observability scan.
