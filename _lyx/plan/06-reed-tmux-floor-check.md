# Card 6 — reed-tmux-floor-check

**Edit:**
- `internal/reedengine#TmuxCmd`
- `internal/reedengine#TmuxCmd.run`
- `internal/reedengine#TmuxCmd.runStdin`
- `internal/reedengine#TmuxCmd.output`
- `internal/reedengine#Engine.ensureServerAndSessionLocked`
- `internal/reedengine/doc.go`

**Create:**
- `plan:internal/reedengine#CheckTmuxFloor` -> `func CheckTmuxFloor(tmuxPath string) error`
- `plan:internal/reedengine#tmuxFloorCache` -> `type tmuxFloorCache struct`
- `plan:internal/reedengine#tmuxFloorCache.check` -> `func (c *tmuxFloorCache) check(tmuxPath string, probe func(tmuxPath string) (string, error)) error`
- `plan:internal/reedengine#tmuxFloors` -> `var tmuxFloors tmuxFloorCache`
- `plan:internal/reedengine#probeTmuxVersion` -> `func probeTmuxVersion(tmuxPath string) (string, error)`
- `plan:internal/reedengine#tmuxVersionProbe` -> `var tmuxVersionProbe = probeTmuxVersion`
- `plan:internal/reedengine#TmuxCmd.checkFloor` -> `func (p TmuxCmd) checkFloor() error`
- `plan:internal/reedengine#setTmuxVersionProbeForTest` -> `func setTmuxVersionProbeForTest(t *testing.T, probe func(tmuxPath string) (string, error))`
- `plan:internal/reedengine#TestTmuxFloor` -> `func TestTmuxFloor(t *testing.T)`
- `plan:internal/reedengine#TestCheckTmuxFloor_RunsNoProbeOnWindows` -> `func TestCheckTmuxFloor_RunsNoProbeOnWindows(t *testing.T)`

**Uses:**
- `plan:internal/toolfloor#CheckTmux`
- `plan:internal/toolfloor#FloorError`
- `internal/reedengine#installFakeTmuxOn`
- `internal/reedengine#reviveTestEngine`

**Intent:** D4, tmux half: every tmux spawn reed makes checks the tmux floor first, once per resolved tmux path in the process, off Windows only.

- `probeTmuxVersion` runs `<tmuxPath> -V` with `exec.Command`, logs the spawn with `logger.Info`, and returns the trimmed stdout or the exec error.
  `tmuxVersionProbe` holds it so a test injects a version line; tests never substitute a tmux binary.
- `tmuxFloorCache` holds a mutex and a map from resolved tmux path to its cached floor answer.
  `check` resolves `tmuxPath` with `exec.LookPath`, keeping the raw path as the key when it does not resolve, and returns a cached answer when one exists; otherwise it probes that path: an exec error is returned wrapped with `%w` and not cached, so the next command probes again and a caller's `errors.Is` on the exec error still holds; a line `toolfloor.CheckTmux` parses is cached with its answer; an unparseable line is logged once with `logger.Warn` naming the line and cached as admitted.
  Bound: only a parsed version below the floor is refused, nothing bypasses a parsed refusal, and the watchdog daemon, a long-lived process reaching tmux through the same runner, keeps its cached answer after a tmux upgrade during its life.
- `CheckTmuxFloor` returns nil on Windows without probing, since psmux has its own version shape and the capability probe's Windows floor; elsewhere it returns `tmuxFloors.check(tmuxPath, tmuxVersionProbe)`.
  It is exported for `reedcli`'s attach (card 7).
- `TmuxCmd` gains an unexported field `floorLine string`, read only when `execHook` is set.
  `checkFloor` returns nil on Windows; with `execHook` set it never reads or fills the process cache: it admits when `floorLine` is empty and otherwise returns `toolfloor.CheckTmux(floorLine)`'s refusal, admitting an unparseable line; without a hook it returns `CheckTmuxFloor(p.tmuxPath)`.
  So hook-driven tests run no probe and keep their exact call counts, such as `lock_test.go`'s zero-call and `overlay_test.go`'s one-call checks.
  Bound: `execHook` and `floorLine` are set only by `reedengine`'s own `_test.go` files, so no production `TmuxCmd` carries a hook, and every production `run`, `runStdin` and `output` reaches `CheckTmuxFloor`.
- `run`, `runStdin` and `output` call `checkFloor` first and return its error before the hook or the spawn.
- The lifecycle server spawn in `ensureServerAndSessionLocked`'s `spawnSession` closure calls `CheckTmuxFloor(e.cfg.Tmux)` before `exec.Command` and returns its error as `start tmux: %w`, so a floor refusal still reads as a failed start and `TestEnsureServerAndSession_RevivalTrigger`'s "start tmux" rows, which run a nonexistent binary, keep passing.
  In production this call is a backstop, not the boot's first refusal: the boot's earlier runner commands over the same `e.cfg.Tmux` path (the capability probe's `-V` and `list-commands`, and the boot's own `has-session` check) refuse a below-floor tmux first from the same cache entry, so the spawn-time check refuses only a spawn no runner command preceded.
  The row "the server spawn refuses before running tmux" reaches it only because a hook runner never reads the process cache.
- The capability probe's own native floor (`minTmuxVersion`, 3.3) stays: the new check refuses a native tmux below 3.4 before the capability probe runs, and psmux keeps its Windows floor there.
- `internal/reedengine/doc.go` describes the check, its cache and the bounds above.
- `setTmuxVersionProbeForTest` replaces `tmuxVersionProbe` and resets `tmuxFloors` for the test, restoring both on cleanup; its godoc says it changes process-global state, so the caller must not be parallel.
  It lives in an untagged `_test.go` file with no OS constraint, so both the off-Windows and the Windows test files below compile against it.
- `TestTmuxFloor`, in an untagged `_test.go` file built off Windows only, is a serial table test (no `t.Parallel`, naming the process-global probe and cache):
  "the runner refuses before running tmux" builds a `TmuxCmd` with a fake through `installFakeTmuxOn`, sets `floorLine` to `tmux 3.3a`, and asserts `run`, `runStdin` and `output` each return a `*toolfloor.FloorError` and the fake saw no call;
  "a hook runner without a floor line runs no probe" installs a counting probe and asserts a hook-driven `run` succeeds with zero probes;
  "a hook runner with an unparseable floor line is admitted" sets `floorLine` to `tmux master` and asserts `run` reaches the fake;
  "the production runner refuses from the process cache" injects `tmux 3.3a`, builds `NewTmuxCmd` over a path that names no binary (no hook), and asserts `run`, `runStdin`, `output` and `CheckTmuxFloor` on that path each return a `*toolfloor.FloorError`, not an exec error, so the hookless branch and the exported check read the cache before any spawn;
  "the probe runs once per resolved path" injects `tmux 3.4` and asserts two `CheckTmuxFloor` calls on one path probe once, a call with a bare name and a call with its `exec.LookPath` result (using `sh`, which every off-Windows host resolves) share one probe, and a call on a second path probes again;
  "a probe that could not run is retried" scripts an exec error then `tmux 3.4` and asserts that `errors.Is` reads the first result as the scripted error, then nil, and two probes;
  "an unparseable line is admitted with a warning" injects `tmux next-3.6` and asserts nil and one `logcapture` Warn naming the line, then a second call returns nil with still one probe and one Warn;
  "the server spawn refuses before running tmux" takes `reviveTestEngine` with a told spawn order and no recorded session (the path `TestEnsureServerAndSession_RevivalTrigger`'s `FirstBootCreates` row takes to the spawn), injects `tmux 3.3a`, and asserts `ensureServerAndSessionLocked` returns an error that `errors.As` reads as `*toolfloor.FloorError`.
  Go starts the package's parallel tests only after its serial ones finish, so no parallel test sees an injected line.
- `TestCheckTmuxFloor_RunsNoProbeOnWindows`, in a `_windows_test.go` file, installs a counting probe that returns `tmux 3.3a` and asserts `CheckTmuxFloor("tmux")` returns nil with zero probes, and that a hookless `NewTmuxCmd`'s `checkFloor` returns nil with zero probes.

**ImpactSummary:** Every tmux command reed runs gains a once-per-path version check ahead of the spawn, invisible to hook-driven tests; callers see a new refusal only on a native tmux below 3.4.
