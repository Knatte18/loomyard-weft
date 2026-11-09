# Machine name from a machine-wide config, not only the hostname

`machine-name-config` · Note · enhancement, config, slack

## Seen (operator, 2026-11-09)

The machine's name comes from one hard-coded rule: `internal/machinename` takes the hostname up to its first dot, lower-cased (`osl-1046.ad.norceresearch.no` reads `osl-1046`).
It is the only production reader of the hostname and has no way to override it.

## Decision (operator, 2026-11-09)

A machine can be given a short name of its own in a machine-wide config file that lym reads, so the name is no longer only a fixed rule.
`osl-1046` is too long to work with and should read `osl46`; the machine `neve` keeps `neve`, which the hostname rule already gives.
A machine with no value in the file keeps the hostname rule, so no machine changes behaviour until it sets one.
This runs before the Slack work that puts the machine name first in channel names.

## What the name reaches, to check one by one

- the owner a task or board entry records and compares (`lym adopt`, the owner verdict, landing and tender), so a changed name must not orphan an owner already written under the old one;
- the cross-machine sender address `<machine>::<sender>` and the Slack channel names built from hub, machine and orch;
- the pair teardown seam that compares machines.

Every caller already takes the name through `machinename.Name`, so the change sits in that package and its config loader.

## Open for Discussion

Where the machine-wide file lives: the per-user state directory beside the hub registry and the Slack credential file, or the existing `machine` layer of `lym config`, which is per hub today.
How a machine whose name was already written into owners is migrated when it takes a new name.
