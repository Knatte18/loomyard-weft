# [.NET 005] Verification protocol for a .NET repo: slow-suite tier, numerical-baseline capture/compare, verify timeouts and logs

`dotnet-verify-protocol` · Note · enhancement, loom, webster, config, dotnet

Imported from [issue #609](https://github.com/Knatte18/loomyard/issues/609).

Superseded shape (orch, 2026-10-20): the synchronous `Capture-Baseline` row below gave way to the async `Job-Start`/`Job-Join` pair of note `regression-baseline-verify` (operator decision 2026-10-17: `Job-Join` right before Webster); this note keeps the Models facts, and the two are meant to merge when the regression tier becomes a task.

## Merged from note `baseline-drift-gate` (2026-10-15)

From `manifest/roadmap.md` Someday.
loomyard captures no before-state, so it cannot tell a pre-existing failure from one a task caused ([millhouse#1130](https://github.com/Knatte18/millhouse/issues/1130)).
In a numerical target repo (the drilling simulator) the failure to catch is drift, which an exit-code verdict like `plan.Verify`'s cannot express.

Agreed shape, not yet a design doc:
- A shared `Capture-Baseline` producer row right after `Preflight`, shared by reference the way `preflightshed` and `landingshed` share theirs, so the planner writes against the current numerical signature.
- The artifact lives in `.lyx/baseline/`, never `_lyx`: a numerical baseline is machine-bound (floating point, BLAS, compiler flags).
  Capture before the first edit, recompute after, compare, discard.
- Two seams, both owned by the target repo: `capture` (tree → artifact) and `compare` (before, after → verdict + findings, with per-channel tolerance).
  Loomyard owns the row, the seam shapes, and the artifact location.
- The gate rides `shuttleengine.GateSpec` on the `Webster` row; registering a validator name in `resolveGateSpec`'s switch is the unbuilt part.

Open:
- Whether compare runs once at Webster's exit (what `GateSpec` gives, since it fires at the Master run's finalize) or per batch (exact localization, a new batch-loop hook, N captures).
- A middle option: gate once, and localize by making the predicate of Webster's integration-stage `bisect` pluggable, at log₂(N) captures; bisect assumes monotone drift.
- A baseline tree whose capture itself fails should be recorded, not halt the run.

## Issue #609

## Summary

NORCE-DrillingAndWells/Models (C#/.NET 9, NUnit) is about to move from Millhouse to LoomYard.
LoomYard's plan-level `## verify:` gate covers this repo's ordinary test suite.
It does not cover the rest of the repo's verification protocol: a gated slow suite, and a **numerical-baseline capture/compare harness** whose reference must be captured *before* the first code change and compared at the end.
This issue describes, as concretely as possible, what the repo needs from the tool, what it does today by hand or through Millhouse, and the failure modes seen in practice.
Items are marked **required** (the repo cannot be developed safely without it) or **wanted**.

## Repo facts the tool has to know

- **Project root is not the git root.** Everything runs from `src/csharp/NORCE.Models/` (it holds `NORCE.Models.sln`, `CLAUDE.md`, `.scratch/`, the task-state dir). Verify commands, scratch paths and baseline paths are relative to that directory, never to the git root.
- **Several task worktrees run at the same time**, under `<container>/wts/<slug>/`, often three to five at once, each building and testing. They share one machine.
- **Plain `dotnet test` on the solution is safe:** the two gated projects set `IsTestProject=false` unless their MSBuild property is passed, so the solution run only runs the fast suite.
- Test namespaces are flat (`NORCE.Models.Tests`), so `--filter FullyQualifiedName~X` matches a **class** name, not a folder.

## The three test projects

| Project | Gate | Runtime (quiet machine, 2026-10-14) | When it must run |
|---|---|---:|---|
| `NORCE.Models.Tests` | none | ~2 min, 1537 tests | every implementation attempt (the plan-level verify) |
| `NORCE.Models.SlowTests` | `-p:RunSlowTests=true` | ~2 min, 21 tests | once at finalization, before the PR |
| `NORCE.Models.NumericalBaseline` | `-p:RunBaseline=true` | ~9 s, 8 fixtures | twice: capture before the first change, verify at finalization |

A full solution build is ~30 s.
Under three parallel worktrees the fast suite was observed at 2.5–3.7 min per run.
The full per-test table is in `src/csharp/NORCE.Models/doc/test-suite-runtime.md` in that repo.

## Required 1: a second must-pass gate tier for the slow suite

The plan-level verify is the fast suite. The slow suite must also pass before the PR, but running it on every gate attempt doubles the attempt cost for no gain.

Needed:
- A verify tier that runs **once, at landing** (before Describe/Publish), not per attempt: `dotnet test NORCE.Models.SlowTests -p:RunSlowTests=true --nologo -clp:ErrorsOnly`.
- Same rerun-once flakiness policy and the same timeout behaviour as the existing gate; failures go back to a fixer the same way.
- Configured per repo (the command lives in repo config, not in each plan), since it never varies between tasks.
- **Zero discovered tests is a failure.** Running a gated project without its property discovers nothing and still exits green. A gate that reads the exit code alone passes vacuously if the flag is dropped. The gate must check that the run discovered tests (e.g. a minimum count, or parse the `Total:` line / the TRX).

## Required 2: numerical-baseline capture/compare as a first-class step

This is the part with no LoomYard equivalent today, and the one most likely to be gotten wrong by an agent.

### What the harness is

`NORCE.Models.NumericalBaseline` runs five purpose-built regression programs plus fixtures and string-compares their scrubbed console output against stored files. It is **not a test suite**: one run proves nothing.

- **Capture:** `dotnet test NORCE.Models.NumericalBaseline -p:RunBaseline=true --nologo`. With no baseline present, every fixture reports `Inconclusive` and writes `<Fixture>.<Method>.txt` plus `_source.txt` (commit, branch, dirty flag, timestamp) into `.scratch/numerical-baseline/` beside `CLAUDE.md`. `.scratch/` is gitignored; baselines are deliberately never committed.
- **Verify:** same command plus `-e NORCE_BASELINE_MODE=verify`. A missing baseline is a hard failure, never an invitation to record one. A deviation fails with the first differing index, the baseline path and the source commit.
- **Side capture for diagnosis:** `-e NORCE_BASELINE_DIR=<absolute dir>` writes the post-change output into a separate directory, so the two directories can be diffed fixture by fixture without touching the trusted one.

### What the tool must do

1. **Capture at the branch point, in the task worktree, before the first code change.** The worktree HEAD at spawn *is* the reference. Capture runs right after the worktree is created (or as the first step of implementation), on the code already checked out.
   - Never check out a second copy to capture from: no `git worktree add`, no clone, nothing under `.scratch/` or the task-state dir. (Millhouse did exactly this once, Millhouse #1181: a nested checkout under `.scratch/verify-baseline-<hash>/`, 8x slower and a corruption risk for the outer repo. It is fixed there; LoomYard must not repeat it.)
   - If code has already changed when the step runs, **do not capture**. Say so and halt: a baseline taken from half-implemented code can only bless drift, never detect it.
2. **Trust check before reusing an existing baseline.** Read `_source.txt`; the recorded commit must be the branch point. The `dirty:` flag is coarse (any untracked file sets it, including the tool's own task-state files), so judge by `git diff <commit> -- '*.cs'` being empty, not by the flag.
3. **Verify at landing**, after the slow tier, with `NORCE_BASELINE_MODE=verify`. A capture-mode run reports every fixture `Inconclusive`; the gate must not read that as a pass. Verify mode reports `Passed`.
4. **On deviation, halt for the operator; never auto-fix and never auto-refresh.**
   - Capture the post-change output into `.scratch/numerical-baseline-post/` with `NORCE_BASELINE_DIR`, diff the two directories, and surface the diff fixture by fixture in plain terms.
   - The operator decides whether the drift is intended. **Intended drift is common:** most physics fixes in this repo move numbers on purpose, and the PR then explains the drift per fixture.
   - The tool must **never delete or overwrite a baseline file to make a comparison pass.** A refresh is an explicit operator command, and it backs up the old directory (e.g. `.scratch/numerical-baseline-pre-refresh/`) before recapturing.
5. **Carry the outcome into the change description**: "no drift" or the per-fixture drift summary, as a check item for the PR reviewer, alongside the verify-gate triage sections.
6. **Optional per-batch drift snapshots** (wanted, not required): capturing into `.scratch/numerical-baseline-after-batch-N/` after each batch made it possible to say which batch moved which fixture. Cheap (~9 s) and diagnostic only; never a gate.

### Why this matters

The fast and slow suites assert physics properties; they do not detect a silent numerical shift. The baseline is the only instrument that does. A gate that runs it once, or captures at the wrong moment, is vacuous and looks green.

## Required 3: verify timeouts and observable verify runs

Observed under Millhouse on 2026-10-13: the background baseline pre-flight (which ran the fast suite once, ~2 min when run directly) took **87 min** in one task, 22 min (three commands) in another, 7 min in a third. The cause is unknown, because the log lived inside the worktree and was gone after teardown, and the run had no timeout (`baseline_verify_timeout_minutes: null`). Direct reproduction attempts (pipe capture of `dotnet test` and of a full rebuild) closed normally.

Needed (LoomYard already fails a timed-out gate without rerun; these are the surrounding pieces):
- **A default timeout** on every verify run, set per tier in repo config (fast ~10 min, slow ~10 min, baseline ~3 min here), never "no ceiling" by default.
- **Per-run timing in the log:** command, cwd, start, end, wall seconds, exit code, discovered/passed/failed counts.
- **Logs that survive teardown**, in the reports directory or the archive, so a slow run can be diagnosed after landing.
- Wall time of each verify run listed in the run's final summary, so a regression in suite runtime is visible without digging.

## Wanted 4: fewer full-suite runs per task

One task (eight fixes, several batches) ran the full fast suite about 15 times, ~35 min, through implementer, fixer and review rounds. LoomYard's single plan-level verify per attempt already helps. Beyond that:
- Let an implementer's in-batch check be **scoped** (`--filter FullyQualifiedName~<Class>`) and keep the full suite to the gate.
- **Skip the gate for a verified tree:** LoomYard's verified-tree record already does this; it should also cover the slow tier and the baseline verify, keyed on the tree hash, so a Describe/landing step after a passing gate reruns nothing.
- **Doc-only diffs skip all test tiers**: when the diff touches only `*.md` (no `*.cs`, `*.csproj`, `*.sln`), no suite runs.

## Wanted 5: exclusivity for timing work, and only for it

Timing measurements (BenchmarkDotNet runs, before/after tables recorded as deliverables) are invalid if any other worktree is building or testing. Correctness runs are fine in parallel, only slower.
- A container-wide **exclusive lock** a task can take for a timing run, which other tasks' verify runs wait on (and only those, not their agents).
- Never serialize correctness runs.

## Wanted 6: one build, several tiers

The tiers can share one build: `dotnet build NORCE.Models.sln -p:RunSlowTests=true -p:RunBaseline=true`, then each tier with `--no-build`. This saves ~30 s per extra tier at landing. If the tool builds once, the gate properties must be passed to the build too, or the gated projects build as non-test projects and discover nothing (see Required 1, zero-tests-is-failure).

## Repo rules a reviewer rubric should be able to check

Not tool features, but things a configurable review checklist should carry for this repo:
- No test in any of the three projects runs, mirrors or references a `Program*` demonstration (`NORCE.Models.Program`). Check with a grep of the test projects for `Program0NN_`.
- Comparisons against stored output live in `NORCE.Models.NumericalBaseline` alone; an in-process test may byte-compare two arms it builds itself.
- No argument guards (`ThrowIfNull`, range checks) and no try/catch around the repo's own code.

## Acceptance

A LoomYard task in this repo, run end to end, without the agent being told any of the above:
1. captures the numerical baseline in its own worktree before the first code change, with `_source.txt` naming the branch point;
2. runs the fast suite as the plan-level gate, with a timeout and logged timing;
3. at landing, runs the slow suite once and the baseline in verify mode, refusing zero-test and Inconclusive outcomes;
4. on baseline drift, halts with a per-fixture diff and waits for the operator, never touching the baseline;
5. puts the drift outcome and each tier's wall time in the change description;
6. leaves its verify logs where they survive teardown.


## Issue comment, 2026-10-14

## Addendum: run the pre-implementation captures detached

Everything that must run on the branch point can start **detached at spawn** and run while discussion and planning go on. No code changes until implementation starts, so the tree is the right reference for the whole window. Two runs qualify:

- the numerical-baseline capture (`dotnet test NORCE.Models.NumericalBaseline -p:RunBaseline=true --nologo`, ~10 s plus build);
- any pre-existing-failure reference for the verify gate (the fast suite once, ~2 min), if LoomYard keeps one. It can also skip this when the parent tip is already a verified tree.

Requirements on the detached job:

1. **It starts at spawn, on the spawn HEAD**, and records that commit. Discussion and plan write only task-state files, which do not affect the run. The `_source.txt` dirty flag will be set by those files, so the trust check compares `*.cs` against the recorded commit, not the flag.
2. **Implementation waits for it before the first code edit.** A batch must not start while the capture is running: an edit mid-capture gives a reference that is part old code and part new. Normally the job finished long before, so the wait costs nothing.
3. **A failed or timed-out capture halts before implementation.** It never falls back to capturing later, because a capture after the first edit can only bless drift.
4. **It has the timeout and per-run log from Required 3**, and the log survives teardown. The 87-minute case was exactly this kind of detached pre-flight job, with no ceiling and a log that disappeared with the worktree.
5. **It builds with the gate properties** (`-p:RunBaseline=true`, plus `-p:RunSlowTests=true` if the slow tier shares the build), and it does not hold a lock that blocks the agent's own work while it runs.

With that, verification adds nothing to the critical path before implementation. The only verification time left on the critical path is the gate after each attempt and the landing tiers.