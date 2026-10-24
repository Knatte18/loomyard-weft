# [.NET 004] Asynchronous test tiers in shed: the numerical baseline captured before implementation, compared at the final gate

`regression-baseline-verify` · Note · feature, shed, webster, batten, config, dotnet

## Seen (operator, 2026-10-17)

The operator's C# work repository has a regression tier: a suite run before the implementation writes profiles, and the same suite run after it compares against them to catch drift.
Many tasks there are pure refactorings that must not change behaviour, and this tier is what proves it.
Operator view, not a decision: regression tests are not always wanted, since some tasks expect drift, for example when a model is changed; this must be handled, perhaps by a flag on the task in the board.

## Orch recommendation, not a decision

- A per-task setting in the board entry's `run_settings`, with three values:
  - `off`: no baseline, no compare.
  - `preserve`: any drift fails the final gate, as a verify failure.
  - `expect-drift`: baseline and compare still run, but the drift report goes to the reviewers and the alignment agent, who judge whether it matches the task's goal; drift alone fails nothing.
  The hub's default lives in config.
- The hub declares the tier in `language.yaml` as three commands: the profile generator, the compare (the repository's own tool, with its tolerances) and the profile directory. lyx only orchestrates and knows no format.
- Operator view, not a decision: the baseline belongs in shed as a producer; it often takes long, so it should start at the beginning as an async command while Discussion and the rest run.
- Orch recommendation, not a decision: shed runs one producer at a time today, so async becomes a generic shed pair of ordinary producers, usable by any recipe:
  - `Job-Start` (a side action with a config flag) starts a detached job, records its handle in run state and returns at once.
  - `Job-Join` waits for the job and fails with its log when it failed.
- Operator view, not a decision: the baseline runs in the task worktree itself, since loom changes no code before Webster, and its output goes to a gitignored place such as `.scratch/`; large simulation time series are never committed.
- Operator decision 2026-10-17: `Job-Join` sits right before Webster, the first row that changes code, so Webster never edits or builds (`bin/`, `obj/`) while the baseline still runs; when Discussion and Plan take longer than the baseline, the join costs nothing.
- Operator view, not a decision: fixed rows in loom for a case that may never come feel clumsy, but each only checks its flag and moves on in milliseconds when it is off.
- Precedent: batten's Open-Window, Seed-IDE and Open-IDE are such rows today, each with an `entry_gate` on a `batten.yaml` flag read from the pair's config stack (shed skips the row and follows `on_done` when the gate is false), so a task's `run_settings` sets it per task. Unlike those, `Job-Join` must be able to fail the run, since those rows only warn.
- The job takes a gate slot like every test run, and obeys a hub pause like the run's other work (note `pause-background-work`).
- Batten tears the job down with the run.
- The compare runs at the final gate only, never per card, to keep the cost down.

## Stations (operator view, not a decision, 2026-10-17)

Loom's recipe could hold fixed command boxes of one type, each taking several entries set per repository:

- Before Discussion: a box that starts the async regression suite, which writes the baseline. The IDE and terminal rows may be their own box of the same type; they stay in any case.
- Right before Webster: a box that only waits for the regression suite to finish.
- Before Finalize: a box that checks whether the regression passed. What it does when it did not is the open hard part.

## On a failed compare (orch recommendation, not a decision)

- Place the compare right after the Webster review (before Describe and Merge-In), not before Finalize: the baseline and the code then share one parent, so drift from other tasks landed meanwhile never enters the compare, which settles the parent-move question.
- Operator view, not a decision: a jump back to `Webster-Burler-Review` on a failed compare is jumping around, and not clean.
- `preserve`: the compare is its own perch, built like every other perch: a check row that runs the compare command, and a fix row that gets the drift report as its finding and hands back to the check. It loops under its own caps and escalates to the orch past them, with no jump into another segment.
- Operator view, not a decision (stated as very important): when regression is not needed, all of it is skipped completely.
  Orch recommendation, not a decision: with `off`, or no regression tier in the hub's `language.yaml`, every regression row is a no-show that reads its setting and is Done: no job starts, no slot is taken, no build runs and no agent is spawned; a hub with no regression tier never shows these rows in its status.
- Operator view, not a decision: a perch in loom's main line starts whatever happens, so it is not skipped by itself.
  Orch recommendation, not a decision: skip it the way darn skips its review perch today: the perch's first row carries an `entry_gate` (darn's Describe row has `entry_gate: budget.Code-Review`), so with a budget of 0 shed records a `skipped` entry and follows `on_done` past the perch (see `contracts/recipes/darn-recipe.yaml`). With regression off for the task or the hub, the regression perch's first row gets an `entry_gate` on the setting the same way, and the task's `run_settings` sets it per task.
  The perch's check row runs the compare command, not an LLM judge, so even when it is entered no agent starts until the compare fails.
- Operator view, not a decision: a compare that finds no drift is skipped onward too.
  Orch recommendation, not a decision: under `preserve` the check row passes and the fix row never runs, so no agent is spawned. Under `expect-drift`, no drift is itself worth a line to PR-Align, since a model change that moves no numbers may not have taken effect.
- `expect-drift`: the compare fails nothing; the drift report goes to PR-Align, where the alignment agent judges it against the task's goal and escalates when it cannot tell.

## Open

- How long the generator takes on the work machine, and whether it can be scoped like the other tiers.
- Whether `preserve` should also run the compare per batch, to point at the card that caused drift.
- A parent that moves during the run is merged in at Merge-In, after the baseline was taken: drift from other landed tasks then shows in the compare. Re-take the baseline at the new parent, or accept it as a known limit.

## Operator decision (2026-10-21): asynchronous tiers in shed

The baseline is the hub's asynchronous test tier, declared in the tier config of task `test-tiers`.
Shed gets a row that starts an asynchronous tier in the background and records the job, and a later row that waits for it to finish; the rows between them run as usual.
The pre-implementation capture on the parent (the detached capture under Models baseline harness) runs through the same pair of rows.

## Related

Task `test-tiers` and task `dotnet-scoped-verify`, which took in note `dotnet-verify-protocol` but left its baseline part here.

## Models baseline harness (from note `dotnet-verify-protocol`, issue #609, 2026-10-24)

`NORCE.Models.NumericalBaseline` runs five purpose-built regression programs plus fixtures and string-compares their scrubbed console output against stored files; one run proves nothing.

- **Capture:** `dotnet test NORCE.Models.NumericalBaseline -p:RunBaseline=true --nologo`, run from `src/csharp/NORCE.Models/`. With no baseline present, every fixture reports `Inconclusive` and writes `<Fixture>.<Method>.txt` plus `_source.txt` (commit, branch, dirty flag, timestamp) into the gitignored `.scratch/numerical-baseline/`; baselines are never committed.
- **Verify:** the same command plus `-e NORCE_BASELINE_MODE=verify`. A missing baseline is a hard failure; a deviation fails with the first differing index, the baseline path and the source commit.
- **Side capture for diagnosis:** `-e NORCE_BASELINE_DIR=<absolute dir>` writes the post-change output into a separate directory, to diff fixture by fixture.

What the tool must do:
1. Capture at the branch point, in the task worktree, before the first code change; never from a second checkout (Millhouse #1181). If code has already changed, do not capture: halt.
2. Before reusing a baseline, check that `_source.txt`'s commit is the branch point by `git diff <commit> -- '*.cs'` being empty, not by the dirty flag.
3. Verify at landing in verify mode; a capture-mode `Inconclusive` is never a pass.
4. On deviation, halt for the operator with a per-fixture diff; never auto-fix, never delete or overwrite a baseline. Intended drift is common; a refresh is an explicit operator command that backs up the old directory first.
5. Carry "no drift" or the per-fixture drift summary into the change description.
6. Wanted: per-batch drift snapshots, diagnostic only, never a gate.

Detached capture: it starts at spawn on the spawn HEAD and records that commit; implementation waits for it before the first edit; a failed or timed-out capture halts before implementation and never falls back to a later capture; it has a timeout and a log that survives teardown; it builds with the gate properties and holds no lock that blocks the agent.

## Operator view (2026-10-24): the compare is an ordinary tier

The regression compare is a plain test tier of the hub, such as Models' `tier2` (`tier0` unit Tests, `tier1` SlowTests), run by the gate like any other tier.
That the capture ran before the implementation is irrelevant to the tier: it only reads a baseline that exists.
A failing compare is then an ordinary red tier, fixed by the gate's usual fixer, so the separate compare perch and its rows above are not needed.
Which gates run which tiers is the hub's own `gate.yaml`, set per repository; nothing here pins it.
The capture before implementation stays this note's async job.

A compare has three outcomes:

- byte-identical: passes;
- small drift of an acceptable size: passes;
- total drift: those regression tests are not run at all, for example when the task changes the model on purpose.

Open: how "small drift" is judged; it needs tuning and likely judgement, perhaps an LLM that judges whether a result is close enough.
