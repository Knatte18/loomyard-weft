# Gate and agents survive task-limit pressure; tier dir follows anchor

This change addresses issues #885, #887, #888 and #884: `lym gate test` and the agents lym spawns hold up under process-limit pressure, a gate tier with no `dir` runs in the task's anchor, and a failed verify gate's findings name what failed.

## What changes

- **No lingering MSBuild servers (#885).**
  `proc.BuildServerOffEnv` declares `MSBUILDDISABLENODEREUSE=1` and `DOTNET_CLI_USE_MSBUILD_SERVER=0`.
  Reed's pane launch line exports them in every strand pane, after the `lym` prelude and before the name exports, and keeps them when the prelude is dropped.
  `lym gate test` sets them in the environment of every tier, build, vet and guard run, replacing any inherited value.
  A tier's or build's own `env` still applies after them.
  `docs/dotnet-hub.md` loses its "warm build processes" stance and describes this instead.
- **A resource-shortage failure is rerun once (#887).**
  When a tier run fails without timing out and its output holds `resource temporarily unavailable`, `failed to create new OS thread` or `cannot fork` (in any case), the gate prints a line naming the marker and reruns the tier once inside the slots it already holds, after a ten-second delay.
  The rerun's result is the tier's result, and its log opens with that line.
  A rerun that fails on a marker again keeps its exit code, and the closing `gate test: failed runs:` line labels it a resource shortage, an environment error, with the way forward of re-running once the machine's process load drops.
- **An unset `dir` follows the anchor (#888).**
  `gateslot` no longer fills an unset tier or build `dir` with `.`, and the new `Config.WithDefaultDir` fills every empty one.
  `lym gate test` calls it with the worktree's `AnchorRel` right after loading the config.
  A hub anchored at `dotnet/` therefore runs `dotnet test LymCsharp.sln` in `dotnet/`, while a set `dir` stays relative to the worktree root.
  A worktree anchored at its root gets `.` as before, so its tier fingerprints and green entries do not change.
- **Verify findings lead with the failures (#884).**
  `loomshed`'s verify failure findings run `websterengine.ParseVerifyFailures` over the log and list each failing test and package under `Failures:` with its own output.
  For a test whose output `lym gate test` rendered above its `--- FAIL` header, as `go test -json` output is rendered, the lines above the header are quoted.
  The gate's closing failed-runs line follows, found by the new shared `gateslot.FailedRunsPrefix`.
  The section is bounded at 8 KiB after the first failure.
  A log that names no failing test or package keeps today's log tail.

## Design decisions

- The build-server variables live in `internal/proc` because reed and gatecli both import it already, and one declaration keeps the two sites in step.
- Only an unset `dir` follows the anchor.
  Making every `dir` anchor-relative would move every hub's set `dir`, and would break a hub that already spells `dir: dotnet` under a `dotnet/` anchor.
- The shortage retry is bounded to one rerun per tier run.
  A test whose own output holds a marker costs one extra run and keeps its verdict.
  The gate verdict is never marked transient (`PATTERN-transient-stop`).
  The build step is not rerun.
- The dobby holding new run starts near the user slice's task limit, offered as optional in #885, is left out of this change and filed as board note `dobby-task-limit-hold`.
- EAGAIN retries in fabricengine's git wrapper and transient loom halts belong to board entry `transient-halt-recovery` and are untouched.

## Tests

- `TestComposePaneLaunchLine` and the spawn launch-script rows assert both exports in each dialect, including when the executable cannot be resolved.
- The gate scenario's env row starts the gate with the opposite values and asserts that the tier sees `1` and `0`.
- `TestGateTest_UnsetDirFollowsTheAnchor` runs a `backend`-anchored hub: the build and a tier with no `dir` run in the anchor, and a tier with `dir: .` runs in the worktree root.
- `TestRetryOnShortage` covers the no-rerun cases (pass, failure without a marker, timeout, refusal, cancelled call) and the rerun outcomes (pass, ordinary failure, shortage twice, shell fork failure).
  `TestFailedRunLabel` pins the closing-line label.
- `TestVerifyFailureFindings` gains a row with failures early in a log that ends in passes, with output both above and under the headers, and asserts that no tail is quoted.
