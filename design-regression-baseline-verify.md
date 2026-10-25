# Regression as three Drift rows: Drift-Baseline after Discussion, Drift-Await before Webster, Drift-Verify after the Code perch

`regression-baseline-verify` · Task · feature, shed, webster, batten, config, dotnet · running · Plan-Bouncer

## Seen (operator, 2026-10-17)

The operator's C# work repository has a regression suite: run before the implementation it writes profiles, and run after it compares against them to catch drift.
Many tasks there are pure refactorings that must not change behaviour, and this suite is what proves it; other tasks change a model and expect drift.

## Design (operator, 2026-10-24)

- Regression is not a test tier: no gate runs it, it is never narrowed or carried, it has two commands (capture and compare), and its outcome is a drift report for an agent, not pass or fail.
  It gets its own config block in the module that owns the Drift rows: a hub flag turning it on, the capture command, the compare command, how a skip list of tests is passed in, and the models of its agents.
  Its commands still take a gate slot, so it never runs beside other heavy test runs.
- Experience from Models: unwanted small drift is beyond Webster's Sonnet implementer; finding why takes a stronger agent with overview, tracing and logging and many varied test runs.
- Discussion knows nothing of regression; its stencil stays the same for every hub.

Loom gets three rows, each a no-show that spawns nothing when the hub's flag is off:

1. `Drift-Baseline`, after Discussion: a Sonnet agent reads the decision record and decides the task's regression scope: off (the task changes the model on purpose), partial (with the tests that will drift totally, which are not run), or full (a pure refactoring, everything byte-identical), and writes down the drift it expects.
   When regression is on, it starts the baseline capture in the background in the task worktree and moves on, so Plan runs meanwhile.
2. `Drift-Await`, right before Webster: mechanical, no LLM; it waits until the capture is done, since no code may change before the baseline exists.
   Plan takes far longer than the regression suite, so it rarely waits.
   A capture in a separate checkout was rejected: a .NET build in a fresh worktree takes too long.
   A failed or timed-out capture halts the run before Webster; it never falls back to a later capture.
3. `Drift-Verify`, after Webster and the Code perch, before Describe and Parent-Align: runs the compare mechanically; when it passes, no LLM is spawned.
   When it fails, it spawns one strong agent (Opus or stronger, a config setting) that, within a budget, judges which drift is acceptable against the task and `Drift-Baseline`'s expectation, finds the cause of the rest and fixes it until the compare passes; it writes its reasoning down and escalates to the orch when unsure or past its budget.
   One agent does both judging and fixing; it is no perch.

Darn: `Drift-Baseline` runs before Darn-Write, reads the board entry instead of the decision record, and runs the capture to the end before Darn-Write starts, so darn has no `Drift-Await`; `Drift-Verify` runs after Darn-Write and before Code-Review.

## Operator decisions (2026-10-25)

- The task adds three new producers and wires them into loom and darn; it changes no existing producer's behaviour.
- One clearly named config key turns regression on, off by default; this hub, LoomYard, keeps it off.
- The Drift rows are not hidden: with the hub's flag off they are skipped like the PR rows are today when PRs are off, with no recipe overlay.
- `Drift-Verify`'s own code changes most likely need no review; if one is added, it is a single solo agent, never a perch.
- No drift report type of its own: no format is pinned in `contracts/specs/`; the agent reads the compare tool's own output.
- Models lives on OSL-1046, not on this machine: the task builds and tests everything it can here without Models, and the Models side follows there.

## Models baseline harness (issue #609)

`NORCE.Models.NumericalBaseline` runs five purpose-built regression programs plus fixtures and string-compares their scrubbed console output against stored files.

- **Capture:** `dotnet test NORCE.Models.NumericalBaseline -p:RunBaseline=true --nologo`, run from `src/csharp/NORCE.Models/`. With no baseline present, every fixture reports `Inconclusive` and writes `<Fixture>.<Method>.txt` plus `_source.txt` (commit, branch, dirty flag, timestamp) into the gitignored `.scratch/numerical-baseline/`; baselines are never committed.
- **Compare:** the same command plus `-e NORCE_BASELINE_MODE=verify`. A missing baseline is a hard failure; a deviation fails with the first differing index, the baseline path and the source commit; a capture-mode `Inconclusive` is never a pass.
- **Side capture for diagnosis:** `-e NORCE_BASELINE_DIR=<absolute dir>` writes the post-change output into a separate directory, to diff fixture by fixture.
- Before reusing a baseline, check that `_source.txt`'s commit is the branch point by `git diff <commit> -- '*.cs'` being empty, not by the dirty flag.
- A baseline is never deleted or overwritten by a run; a refresh is an explicit operator command that backs up the old directory first.

Note: `carry: false` from `dotnet-scoped-verify` was built with regression in mind; the timing tier still needs it.
