# A killed seed leaves board sync naming a false, looping way forward

`seed-killed-way-forward-loops` · Task · bug, fabric · running · Drift-Baseline

## What was seen

Round 3 of `harden-fabricengine` killed the seeding `lym` with `kill -9` while a `pre-receive` hook slept on one ref.
Origin then held the whole seed, yet `lym board sync` refused with "has no upstream in a one-repository hub, so it was never seeded onto origin" and named `lym fabric clone --single-repo ... --reset` (`internal/fabricengine/pushclassified.go`).
The sentence is false, and a one-URL clone onto an origin that already holds a namespaced board is refused, so the way forward may loop.
The `--reset` path was not driven, since a task session may not run `lym fabric clone`.

A related wording gap: a killed seed push that took the records branch and withdrew it says nothing of the withdrawal, where the declined-board message does.

## Idea

Make the message read origin before it claims the board was never seeded, and name a way forward that works when origin holds the seed.
Drive it in the second hardener campaign, after `harden-sandbox-fabric-verbs` lands.
