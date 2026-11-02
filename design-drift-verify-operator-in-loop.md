# Drift-Verify with the operator in the loop

`drift-verify-operator-in-loop` · Task · feature, drift · done

Imported from [issue #14](https://github.com/Knatte18/loomyard/issues/14).

Request from a hub operator (NORCE Models, a numerical simulation repository).

Today Drift-Verify is one autonomous agent: it reads the compare log and the Drift-Baseline report, accepts drift it judges intended with `lym drift accept`, fixes the rest, and when unsure stops for the run's parent. In a numerical code base, whether a drift is acceptable is a judgment the operator wants to make: how large a deviation is acceptable, in which quantities, and whether a change in solver output is a fix or a regression. An autonomous accept can bless a regression.

Wanted: a mode, in the hub's drift config (as `discussion_interactive` is for Discussion), where the last drift step is an interactive session with the operator in the task's worktree. The agent presents the drift per test and per quantity, together with the baseline agent's expectation, discusses it with the operator, and records each acceptance or fix the operator decides, before the run goes on. Drift-Baseline stays autonomous.

The hub's drift config shows no such key today (`enabled`, `capture`, `compare`, `baseline_model`, `verify_model`, `verify_max_runs`).