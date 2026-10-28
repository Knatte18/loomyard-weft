# Gate and agent runs survive task-limit pressure and run in the anchor

`lym gate test` and the agents lym spawns now hold up under transient process-limit pressure, and a gate tier with no `dir` runs in the task's anchor.
A failed verify gate's findings name the failing tests instead of a tail of passing ones.

Closes #885
Closes #887
Closes #888
Closes #884

## What changes

- **No MSBuild leftovers (#885).**
  `internal/proc` declares `MSBUILDDISABLENODEREUSE=1` and `DOTNET_CLI_USE_MSBUILD_SERVER=0`.
  Reed exports both in every strand pane's launch line, and `lym gate test` sets them for every tier, build, vet and guard run, overriding an inherited value.
  A tier's own `env` can still override them.
- **Resource shortage is not a test failure (#887).**
  A tier that fails, without a timeout or cancellation, and whose output holds `resource temporarily unavailable`, `failed to create new OS thread` or `cannot fork` (case-insensitive) is rerun once after a short delay.
  The retry's result is the tier's result.
  A second failure with a marker keeps its exit code, and the closing failed-runs line labels it a resource shortage with the way forward of re-running once the machine's process load drops.
- **Tier `dir` follows the anchor (#888).**
  `gateslot` leaves `Tier.Dir` and `BuildStep.Dir` empty when gate.yaml sets none, and `Config.WithDefaultDir` fills the empty ones.
  `lym gate test` calls it with the anchor path relative to the worktree root right after loading the config.
  A set `dir` stays relative to the worktree root.
  For a worktree anchored at its root the filled dir is `.`, so existing tier fingerprints and green entries are unchanged.
- **Verify findings lead with failures (#884).**
  `verifyFailureFindings` in `internal/loomshed` runs `websterengine.ParseVerifyFailures` over the log.
  When it names a test or package failure, the findings list each with its own output tail under a `Failures:` heading, bounded in bytes, instead of the log tail.
  When it names none, the log tail stays.

Package docs, `docs/dotnet-hub.md` and the `gateslot` template are updated to match.

## Design decisions

- The retry is bounded to one extra run per tier per call and runs inside the slots the tier already holds.
  It marks nothing transient, so a gate verdict keeps its meaning.
  It is logged through `internal/logger`, and its delay is injected through `testRequest` so tests do not wait it out.
- Filling `dir` once in the config means every reader (gatecli, the green record's module key, the dotnet scoper) sees the anchor without its own fallback.
- A test whose own output contains a marker string costs one rerun per call; its verdict is unchanged.
- The first attempt's output stays in the console but not in the run record's log, which holds the retry's output.

## Not covered

- Exported variables reach only shells reed starts; an operator shell is untouched.
- Holding a new run start while the user slice nears its task limit (the second half of #885) is not built.
- EAGAIN retries in fabricengine's git wrapper and transient halts of a loom run belong to `transient-halt-recovery`.

## To check by hand

- Run `lym gate test` in a worktree anchored at a subdirectory (such as the C# hub's `dotnet/`) with a tier that sets no `dir`.
- Start a .NET build from a reed strand pane and confirm no `MSBuild.dll /nodemode:1 /nodeReuse:true` process stays behind.
