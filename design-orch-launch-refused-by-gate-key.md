# A stale gate key refuses the orch launch

`orch-launch-refused-by-gate-key` · Task · bug, claude, gate, config, orch · running · Describe

## What was seen (2026-11-22)

The hub gate.yaml held a key the hardener rebuild (2dbfdc31) retired, `gates.harden-live-copies`, with no `scope`.
`lym s` then failed to resume the orch session: claudeengine refused to prepare the orch launch over that one gate key, so the hub had no orch until the key was removed by hand.
`lym config` has no verb to remove a key, so only the interactive editor could fix it.

## Idea

- An orch or other session that runs no gate should not refuse its launch over a gate key it never uses; warn and launch.
- The gate reinterpretation that retires `harden-live-copies` should also cover an entry with no scope.
- Give `lym config` a non-interactive way to remove a key.
