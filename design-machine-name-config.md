# Machine name from a machine-wide config, not only the hostname

`machine-name-config` · Note · enhancement, config, slack

## Seen (operator, 2026-11-09)

The machine's name comes from one hard-coded rule: `internal/machinename` takes the hostname up to its first dot, lower-cased (`osl-1046.ad.norceresearch.no` reads `osl-1046`).
It is the only production reader of the hostname and has no way to override it.
The operator wants a short name of their own on a machine (`osl46`) and wants lym to read it from a machine-wide config file, not from a fixed rule.

## Idea

A machine-wide config value holds the machine's name; when it is absent, the hostname rule stands, so no machine changes behaviour until it sets one.
Every caller already takes the name through `machinename.Name`, so the change sits in that package and its config loader.

What the name reaches, which the work must check one by one:

- the owner a task or board entry records and compares (`lym adopt`, the owner verdict, landing and tender), so a changed name must not orphan an owner already written under the old one;
- the cross-machine sender address `<machine>::<sender>` and the Slack channel names built from hub, machine and orch;
- the pair teardown seam that compares machines.

Open: where a machine-wide file lives (the per-user state directory beside the hub registry and the Slack credential file, or the existing `machine` layer of `lym config`, which is per hub today), and how a rename of an already-used machine name is migrated.
