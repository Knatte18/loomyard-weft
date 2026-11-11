# lym adopt replays a stale record's stops after the operator cleared them

`adopt-stale-record-stops` · Note · bug, fabric

## Seen (2026-11-11, Neve)

After a partial `lym adopt root --drop-local`, every later `lym adopt root` printed the same reconcile stops: pairs already removed with `lym fabric remove` and a run record cleared by `lym cleanup --apply` still showed as `gone` and `needs_replay`, and root stayed held back.
The call followed the unfinished adopt record instead of judging afresh.
`lym adopt --discard`, then `lym adopt root`, went through at once.

## Idea

A new `lym adopt <subject>` call whose stops come from a finished, failed worker judges the hub again instead of replaying the old record,
or names `lym adopt --discard` as the way forward when the stops no longer hold.
