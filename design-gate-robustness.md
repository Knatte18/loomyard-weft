# Gate and agent runs survive transient task-limit pressure and run in the anchor

`gate-robustness` · Task · bug, gate, shuttle, dotnet, batten, loom · done

## Lingering MSBuild reuse nodes exhaust the user's task limit during .NET runs (was note `msbuild-node-reuse-task-limit`)

Imported from [issue #885](https://github.com/Knatte18/loomyard/issues/885).

#### Seen (Neve, 2026-10-28)

With five loomyard runs and one darn run in the C# sandbox hub (lym-csharp) live, the user slice hit its systemd task limit (user-1000.slice pids.max 3000, pids.current 2997).
Every fork then failed with EAGAIN: three batten runs stopped at Run-Shed or Darn-Write, pushes failed (a git credential helper that cannot fork falls back to a username prompt, reported as an auth failure), and Claude Code's own hooks failed with 'Cannot fork'.
18 idle `dotnet MSBuild.dll /nodemode:1 /nodeReuse:true` processes, about 20 threads each, were left behind by `dotnet build` and `dotnet test` calls made by agents outside the gate; the gate's own runs used `/nodeReuse:false` nodes.
`dotnet build-server shutdown --msbuild` did not end them.

#### Workaround

Killed them by hand: `pkill -u <user> -f 'MSBuild.dll .*nodeReuse:true'`, which took the slice from 2940 to 2500 tasks.

#### Expected

lym sets `MSBUILDDISABLENODEREUSE=1` (and `DOTNET_CLI_USE_MSBUILD_SERVER=0`) in the environment of every agent session and gate run it starts, so no .NET build an agent runs leaves worker nodes behind.
Beside that, the dobby could hold a new run start while the user slice is near its task limit, since six parallel runs sit close to the default limit even without the leak.
## lym gate test reports transient fork/thread exhaustion as test failures (was note `gate-eagain-not-test-failure`)

Imported from [issue #887](https://github.com/Knatte18/loomyard/issues/887).

#### What happened

Two `lym gate test --wait` runs failed with `fork/exec ... resource temporarily unavailable` and `runtime: failed to create new OS thread (errno=11)`.
A `go vet` run and one PreToolUse hook failed the same way ("Cannot fork").
The failures hit packages the change did not touch.
`ulimit -u` was far above the live thread count, and a rerun a minute later was green.
The cause looks like short-lived process or thread pressure from other runs sharing the machine.

#### Why it is abnormal

The gate surfaces these as ordinary build or test failures.
The cause is an environment error that a retry clears, and reporting it as a test failure hides that.

#### What to do

Have `lym gate test` recognise fork/exec EAGAIN and thread-creation failures and report them as a retryable environment error, or retry them automatically.
Consider limiting how many gate runs execute concurrently on one machine.

Provenance: task `ide-seed-current-worktree`, note `Darn-Write.md`.

## Gate tier runs dotnet test in the worktree root instead of the anchor subdirectory (was note `gate-tier-dir-follows-anchor`)

Imported from [issue #888](https://github.com/Knatte18/loomyard/issues/888).

#### What happened

In a task whose project anchor is a subdirectory of the worktree (`dotnet/`), `lym gate test --wait` ran the tier0 command `dotnet test LymCsharp.sln ...` in the worktree root.
It failed at once with `MSB1009: Project file does not exist`.
The hub's gate tiers set no `dir`, and the default run directory does not follow the anchor.

The producer agent worked around it by passing `-C <worktree>/dotnet` with an explicit target, after which all tests passed.
The loom's diff-scoped verify very likely runs the same way, since the producer hit the same failure at the handoff gate: the run halted `blocked` with `gate did not pass after 3 attempts` (anomaly `escalation-to-human`).

#### Why it is abnormal

The failure is a gate misconfiguration, not a code defect, so no amount of agent retrying can clear it.
The halt's way forward, `lym loom resume`, gives a fresh verify budget but leaves the gate config unchanged.
The resumed spawn therefore hit the identical failure and halted a second time.

#### What to do

- Make a gate tier's default run directory follow the task's anchor directory when the anchor is a subdirectory of the worktree.
- Failing that, detect a gate that fails identically with a missing-project error and surface it as a configuration problem, rather than as a verify failure that offers `lym loom resume` as the way forward.

Provenance: task `regression-harness`; notes Darn-Write.md, Darn-Write-2.md, loom-halt.md, loom-halt-2.md.

## Verify-gate findings log tail shows only passing tests, hiding the failure (was note `verify-findings-show-failures`)

Imported from [issue #884](https://github.com/Knatte18/loomyard/issues/884).

#### What happened

A producer row's verify gate failed three times in a row, and the run halted `blocked` with anomaly `escalation-to-human`.
Each re-prompt carried a findings text of the form "The verify command failed with exit code 1", a full-log path and a "Log tail".
The tail was the last lines of the `lym gate test` output, and those lines held only `--- PASS` entries, a final `PASS` and `ok` for the last package.
The `--- FAIL` lines sat tens of thousands of lines earlier in the same log (the verify log ran to roughly 57,000 lines, and the first failure sat near line 25,800).
The only hint of failure in the tail was the gate summary line `tier0 computed ... failed 2 ...`, which names no test.

#### Why it is abnormal

The tail exists so the re-prompted agent can see what failed.
With `go test` output interleaved from parallel packages, the last lines are not where failures are, so the agent got a findings text that looks like a pass.
The agent had to open a 57,000-line log and search for `--- FAIL` by itself, and the budget of three attempts was spent re-running with little to act on.

#### What to do

Build the findings text from the failures rather than from the raw tail.
For `lym gate test` output, extract the `--- FAIL` and `FAIL <package>` lines (with a few lines of context each) and put them first.
Fall back to the tail only when no failure marker is found.
Alternatively, make `lym gate test` print a failed-tests section at the end of its output, so any tail contains it.

Provenance: task `hub-reader-creates-lym`; notes `loom-halt.md`, `loom-halt-2.md`.

