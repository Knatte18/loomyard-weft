# Decision record — gate-robustness

## Goal

`lym gate test` and the agents lym spawns keep working under transient process-limit pressure, a gate tier runs in the task's anchor when its `dir` is unset, and a failed verify gate's findings show the failures instead of a tail of passing tests.

## Scope

- In: issue #885, no .NET build an agent session or a gate run starts leaves MSBuild reuse nodes or the MSBuild server behind.
- In: issue #887, a gate tier that fails on fork/exec EAGAIN or a failed OS thread creation is retried once and, when it fails so again, named as a resource shortage with a way forward.
- In: issue #888, a tier and the build step with no `dir` run in the worktree's anchor directory.
- In: issue #884, the verify gate's failure findings put the parsed failing tests and packages first and quote the log tail only when the log names none.
- Out: the dobby holding a new run start while the user slice nears its task limit (#885's second half, a "could"); it is filed as a board note.
- Out: EAGAIN retries in fabricengine's git wrapper and transient halts of a loom run, which the board entry `transient-halt-recovery` owns.

## Decisions

### Build-server variables live in `internal/proc`

Decision: `internal/proc` declares the ordered name and value pairs `MSBUILDDISABLENODEREUSE=1` and `DOTNET_CLI_USE_MSBUILD_SERVER=0`.
Reed's pane launch line exports them in every strand pane beside `LYM_STRAND_NAME`, and `lym gate test` sets them in the environment every tier, build, vet and guard run gets, overriding an inherited value.

Rationale: both reed and gatecli already import `proc`, a strand pane is where every agent session's shell and tools take their environment, and one declaration keeps the two sites in step.

### An unset tier or build `dir` follows the anchor

Decision: `gateslot` leaves `Tier.Dir` and `BuildStep.Dir` empty when gate.yaml sets no `dir`, and `Config.WithDefaultDir` fills every empty one with a given directory.
`lym gate test` calls it with the worktree's anchor path relative to the worktree root right after loading the config, so a set `dir` stays relative to the worktree root and an unset one is the anchor.

Rationale: a C# hub anchored at `dotnet/` declares commands such as `dotnet test LymCsharp.sln` that only work in the anchor, and every reader of the dir in gatecli, the green record's module key and the dotnet scoper included, reads it from the config, so one fill point covers them all.
For a worktree anchored at its root the filled dir is `.`, so the tier fingerprints and green entries such worktrees hold are unchanged.

### A resource-shortage failure is retried once, then named

Decision: when a tier run fails, has not timed out and was not cancelled, and its output holds a resource-shortage marker (`resource temporarily unavailable`, `failed to create new OS thread` or `cannot fork`, matched without case), the gate prints a line naming the marker and runs the tier once more after a short injectable delay.
The retry's result is the tier's result.
A retry that fails with a marker again keeps its failing exit code, and the closing failed-runs line labels that run as a resource shortage with the way forward of re-running once the machine's process load drops.
Bound: at most one extra run per tier per call, so a test whose expected output holds a marker costs one rerun and its verdict is unchanged; the gate's verdict is never marked transient, per `PATTERN-transient-stop`.

Rationale: the failure is an environment error a rerun clears, and a single bounded retry inside the held slots clears it without the caller spending a verify attempt.

### Verify findings lead with the parsed failures

Decision: loomshed's verify failure findings run `websterengine.ParseVerifyFailures` over the log and, when it names a test or package failure, list each identity with its own output tail under a `Failures:` heading in place of the log tail, bounded in bytes.
When it names none, the findings keep today's log tail.

Rationale: `go test` output of parallel packages puts failures anywhere in a long log, the parser already finds them for webster's verify reports, and loomshed already imports websterengine.

## Constraints

- `PATTERN-transient-stop`: the gate retry marks nothing transient and never changes a gate verdict's meaning.
- `PATTERN-spawn-observability`: the retry is logged through `internal/logger`.
- `PATTERN-gate-slot`: the retry runs inside the slots the tier already holds.
- `PATTERN-test-speed`: the retry delay is injected through `testRequest`, so tests do not wait it out.
- Docs land in the same commits: `internal/gatecli/doc.go`, `internal/reedengine/doc.go`, `internal/loomshed` package doc where it describes the findings, and `internal/proc/doc.go`.

## Auto-mode assumptions

- The tier's own `env` may still override the build-server variables, since a hub that sets them knows its build.
- A marker in a retried tier's first-attempt output stays in the run's console output but not in its run record's log, which holds the retry's output.

## Open risks

- A test whose own output contains a marker string is rerun once per call.
- Exported variables reach only shells reed starts; a process started outside reed, such as an operator shell, is untouched.

## Acceptance criteria

- A strand pane's launch line exports `MSBUILDDISABLENODEREUSE=1` and `DOTNET_CLI_USE_MSBUILD_SERVER=0`.
- A gate tier's command sees both variables, also when the caller's environment set other values.
- A tier with no `dir` in a worktree anchored at a subdirectory runs in that subdirectory; a tier with a `dir` runs in that dir under the worktree root; a worktree anchored at its root runs as before.
- The build step with no `dir` runs in the anchor.
- A tier whose first run fails with a marker in its output and whose retry passes ends passed, with the retry line on stderr.
- A tier that fails with a marker twice ends failed with its exit code, and the closing line names it as a resource shortage.
- A tier that fails without a marker is not retried.
- Verify findings over a log with a `--- FAIL` early and only passing lines at the end name the failing test and its output, and carry no log tail.
- Verify findings over a log the parser finds no test or package failure in keep the log tail.

## Notes for the plan writer

- `runGreenAware` in `internal/gatecli/run.go` is where a tier's `runTier` call can be wrapped by the retry.
- `verifyFailureFindings` in `internal/loomshed/verifygate.go` builds the findings.
- `composePaneLaunchLine` in `internal/reedengine/panebin.go` composes the pane's exports.
