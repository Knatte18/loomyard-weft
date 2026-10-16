# Card 6 — `lyx shuttle run --anchor` takes own-window or hidden

**Edit:**
- `internal/shuttlecli#shuttleCLI.runCmd`
- `internal/shuttlecli#TestRunCLI_Run_FlagValidation`

**Intent:** D2's `lyx shuttle run` half: `--anchor` keeps its `own-window` default and accepts `own-window|hidden`, and its help reads `placement: own-window|hidden`.
Any other value, `below-parent`, `top`, `overview`, an unknown or an empty one, is refused among the flag-shape checks, before the prompt file is read and before `c.runner.Run`, with `invalid --anchor %q; want own-window|hidden`, so no run starts.
The check narrows what the flag lets through and opens nothing.

Tests: `TestRunCLI_Run_FlagValidation` gains rows with a valid `--prompt` and `--output-file` and `--anchor` set to `below-parent`, `top`, `overview`, `sideways` and the empty value, each wanting an error containing `own-window`; the test's nil runner proves none starts a run.

**ImpactSummary:** one flag of one CLI verb; `Spec.validate` and every programmatic spawner of shuttle are unchanged.

**Commit:** 6: shuttle: run --anchor takes own-window or hidden
