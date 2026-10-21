# [.NET 002] Test tiers: every test run through lyx gate test, numbered tiers in hub config, computed scope

`test-tiers` · Task · enhancement, gate, config, shed, dotnet · running · Merge-Burler-Fix

Every test run in a hub goes through `lyx gate test`, against numbered test tiers the hub declares in config, with the scope computed by lyx.

## Seen (orch, 2026-10-17)

`lyx gate test` runs only `go test`: it takes a hub gate slot, then runs `go test -p <go_parallel>` over the named packages.
A hub in another language has no gate, so its verify command (for example `dotnet test` in a C# hub) runs outside the slot pool.
Several runs then build and test at once with no limit, which is what the slot pool exists to stop.
The slot pool in `internal/gateslot` is already language-neutral (one file lock per slot); only `lyx gate test` on top of it is Go-specific.

## Operator decisions (2026-10-21)

### All testing through the gate

- Every test run, an agent's own and every verify gate's, goes through `lyx gate test`, so lyx controls how many run at once and can sequence heavy tiers.
- The shell guard refuses raw test commands (`dotnet test`, `pytest`), as it already refuses raw module-wide `go test`.
- The command keeps its name: `lyx gate` is the module, and `test` stays its verb; no rename.

### Numbered tiers in config

- The hub declares its tiers `tier0`, `tier1`, ... in config, each with its command (placeholders such as `{targets}` and `{parallel}`, so lyx hard-codes no tool), its parallelism, a timeout, and a minimum count of discovered tests.
- Parallelism is per tier, in place of the Go-only `go_parallel`.
- A tier never includes a lower one by definition: each gate names the list of tiers it runs, such as card `[0]`, final `[0,1]`, push `[0,1,2]`.
- A tier may declare `covers: [0]` when its command runs a lower tier anyway, so a gate asking for `[0,1]` skips tier0. Go needs this: untagged files always compile, so `-tags integration` runs the untagged tests too.
- A tier may declare it needs more slots or the whole machine, so a heavy tier is sequenced.

### Scope is computed, never spelled

- `lyx gate test` with no targets runs tier0 over the scope lyx computes from the diff against the parent; Go has this in `internal/impactset`.
- `--tier N` picks another tier.
- `--card <id>` takes the scope from the files the plan says the card touches.
- Explicit targets stay possible, as the exception, such as one package while debugging.
- Agents and gates thus get the same scope, and no agent spells a long module list.

### Verify runs are bounded and observable

- Zero discovered tests is a failure: a run that discovers nothing and exits green passes vacuously.
- Every tier run has a timeout from its config, never no ceiling by default.
- Each run logs command, cwd, start, end, wall seconds, exit code and discovered/passed/failed counts, in logs that survive teardown.

## The hubs

- loomyard: tier0 the untagged tests, tier1 `integration`, tier2 `tmux`, tier3 `llm`. `CLAUDE.md`'s push rule is rewritten in tiers.
- NORCE Models: tier0 the fast unit tests, tier1 the slow-test project, tier2 the numerical baseline. Note `dotnet-verify-protocol` holds the repo facts: project root under `src/csharp/NORCE.Models/`, the gated projects' MSBuild properties, and why a plain solution run discovers only the fast suite.
  The baseline tier needs the asynchronous run in note `regression-baseline-verify`, which is not part of this task; here it is declared and run synchronously only.

## Not in this task

- Test selection for C# (note `dotnet-scoped-verify`); a C# tier runs over its whole project set until then.
- Asynchronous tiers in shed (note `regression-baseline-verify`).
- `lyx gate status` (note `gate-status`).
