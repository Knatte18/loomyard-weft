# Slow gate tier's COMMONLIB_REPO differs from the session's, with no warning

`gate-env-override-warning` · Note · enhancement, gate, dotnet

Imported from [issue #157](https://github.com/Knatte18/loomyard/issues/157).

A darn agent measured a tolerance with the slow gate tier.
The tier's `COMMONLIB_REPO` names a different CommonLib checkout than the session's own environment, and the two checkouts give different series.
Neither the prompt nor the gate's report says so; the agent found it only by reading `lym config gate --print`, after measuring against the wrong checkout.

This is abnormal: a gate run silently uses an environment that differs from the session's, and a result from it misleads.
Fix: have `lym gate test` report the tier's env overrides that differ from the session's environment, or list them in the prompt for a gate-driven task.

Provenance: task cuttings-bed-decay-parity, note Darn-Write.md.