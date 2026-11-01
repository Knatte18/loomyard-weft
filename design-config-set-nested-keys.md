# lym config --set cannot write a key inside a nested map, such as a gate tier env

`config-set-nested-keys` · Task · bug, config, gate · running · Code-Bouncer

## Seen (orch, 2026-11-01)

`lym config gate --set tiers.tier0.env.LYM_TEST_TIER=0` was refused: the value landed one level too high and the gate config no longer loaded ("cannot unmarshal !!str `0` into gateslot.tierFile").
`--set` writes only a top-level key or one entry of an open map, and the editor refuses a non-interactive terminal, so an orch cannot change a key inside a gate tier, such as its `env` or `command`, at all.
Workaround for `gate-one-binary-per-package`: the run's test selector falls back to the build tag's tier, and the gate.yaml switch waits for the operator or a later fix.

## Idea

`--set` takes a dotted path to any key the module's schema reaches, nested maps included, and writes a scalar there; a path into a struct field the schema lacks is refused naming the key.
