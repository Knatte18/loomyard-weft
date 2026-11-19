# Darn prompt tells agent to run lym gate test --gate darn-write, which the hub's gate config refuses

`darn-prompt-unknown-gate-key` · Note · bug, stencil, darn, gate

Imported from [issue #156](https://github.com/Knatte18/loomyard/issues/156).

A darn agent was told to run a single test class through `lym gate test --gate darn-write <project> -- <args>`.
The hub's `gate.yaml` has no `darn-write` key and its tiers hold no `{targets}` placeholder, so both forms were refused.
Only `lym gate test --tier 1 -- --filter ...` worked, after the agent dropped the targets.

This is abnormal: the prompt instructs a command form the hub's own gate config cannot satisfy, and the agent spent calls discovering that.
Fix: make the darn prompt name only a form valid for the hub's gate config, or have the prompt builder check the key and `{targets}` against `gate.yaml` and fall back to the `--tier` form.

Provenance: task cuttings-bed-decay-parity, note Darn-Write.md.

## From issue #162

Imported from [issue #162](https://github.com/Knatte18/loomyard/issues/162).

A Darn-Write agent's instructions said to run a needed test class through `lym gate test --gate darn-write`.
The hub's `gate.yaml` declares only `finalize` and `adopt`, so the named gate key does not exist.
The agent fell back to `lym gate test --tier 1 -- --filter ...`.

This is abnormal because a prompt should name only gate keys the hub declares.
Fix: either generate the Darn-Write prompt from the declared gate keys, or declare a `darn-write` key; also have `lym gate test` refuse an unknown `--gate` with a list of declared keys.

Provenance: task bed-erosion-rate-gap, note Darn-Write.md.