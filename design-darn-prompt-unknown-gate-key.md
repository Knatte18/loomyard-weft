# Darn prompt tells agent to run lym gate test --gate darn-write, which the hub's gate config refuses

`darn-prompt-unknown-gate-key` · Note · bug, stencil, darn, gate

Imported from [issue #156](https://github.com/Knatte18/loomyard/issues/156).

A darn agent was told to run a single test class through `lym gate test --gate darn-write <project> -- <args>`.
The hub's `gate.yaml` has no `darn-write` key and its tiers hold no `{targets}` placeholder, so both forms were refused.
Only `lym gate test --tier 1 -- --filter ...` worked, after the agent dropped the targets.

This is abnormal: the prompt instructs a command form the hub's own gate config cannot satisfy, and the agent spent calls discovering that.
Fix: make the darn prompt name only a form valid for the hub's gate config, or have the prompt builder check the key and `{targets}` against `gate.yaml` and fall back to the `--tier` form.

Provenance: task cuttings-bed-decay-parity, note Darn-Write.md.