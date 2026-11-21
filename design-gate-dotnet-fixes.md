# Gate fixes: verdict from parsed results, only declared gate keys in prompts, env override warning

`gate-dotnet-fixes` · Task · bug, gate, dotnet, stencil, darn · running · PR-Review

## Gate verdict from a parsed result, and crash cause in the log

Imported from [issue #163](https://github.com/Knatte18/loomyard/issues/163).

During diagnostic runs of `lym gate test --tier 1 -- --filter ...`:

1. The first run aborted with "Test host process crashed" and the gate log gave no cause. The identical command passed on the next try.
2. With a detailed console logger among the pass-through args, the gate reported "ran no test" although the run's own output showed `Passed: 1`. The verdict line was wrong for that call.

This is abnormal because the gate verdict is what agents rely on; a verdict contradicting the test output, and a crash with no captured cause, force agents to distrust it.
Fix: parse the test result from the TRX or a logger the gate controls instead of console text, so user-supplied loggers cannot change the verdict; and capture the test host's crash dump or stderr in the gate log when the host crashes.

Provenance: task bed-erosion-rate-gap, note Darn-Write.md.
## Darn prompt names a gate key the hub does not declare

Imported from [issue #156](https://github.com/Knatte18/loomyard/issues/156).

A darn agent was told to run a single test class through `lym gate test --gate darn-write <project> -- <args>`.
The hub's `gate.yaml` has no `darn-write` key and its tiers hold no `{targets}` placeholder, so both forms were refused.
Only `lym gate test --tier 1 -- --filter ...` worked, after the agent dropped the targets.

This is abnormal: the prompt instructs a command form the hub's own gate config cannot satisfy, and the agent spent calls discovering that.
Fix: make the darn prompt name only a form valid for the hub's gate config, or have the prompt builder check the key and `{targets}` against `gate.yaml` and fall back to the `--tier` form.

Provenance: task cuttings-bed-decay-parity, note Darn-Write.md.

### From issue #162

Imported from [issue #162](https://github.com/Knatte18/loomyard/issues/162).

A Darn-Write agent's instructions said to run a needed test class through `lym gate test --gate darn-write`.
The hub's `gate.yaml` declares only `finalize` and `adopt`, so the named gate key does not exist.
The agent fell back to `lym gate test --tier 1 -- --filter ...`.

This is abnormal because a prompt should name only gate keys the hub declares.
Fix: either generate the Darn-Write prompt from the declared gate keys, or declare a `darn-write` key; also have `lym gate test` refuse an unknown `--gate` with a list of declared keys.

Provenance: task bed-erosion-rate-gap, note Darn-Write.md.
## Gate tier env overrides that differ from the session

Imported from [issue #157](https://github.com/Knatte18/loomyard/issues/157).

A darn agent measured a tolerance with the slow gate tier.
The tier's `COMMONLIB_REPO` names a different CommonLib checkout than the session's own environment, and the two checkouts give different series.
Neither the prompt nor the gate's report says so; the agent found it only by reading `lym config gate --print`, after measuring against the wrong checkout.

This is abnormal: a gate run silently uses an environment that differs from the session's, and a result from it misleads.
Fix: have `lym gate test` report the tier's env overrides that differ from the session's environment, or list them in the prompt for a gate-driven task.

Provenance: task cuttings-bed-decay-parity, note Darn-Write.md.
