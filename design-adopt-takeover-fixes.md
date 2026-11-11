# Adopt takes over without hanging behind a pause, failing on lym files or replaying stale stops

`adopt-takeover-fixes` · Task · bug, fabric, tender · running · Drift-Verify

Fix the three adopt faults below in one run; each note's Idea is the starting point.

## From note adopt-held-by-pause

## Seen (2026-11-11, Neve)

A hub-wide `lym tender pause` set before the operator moved to another machine was still live when they came back.
`lym adopt root` then hung: its worker's first network call logged "held by a hub-wide pause" and waited with no word to the operator, whose terminal showed nothing.
Lifting the hub entry alone (`lym tender resume hub`) let it go on; the deferred records push that lift spawned then held the weft push lock,
and the adopt's origin reconcile stopped on `main-weft (busy)` after it had already set aside two pairs.

## Idea

An adopt is the operator's own takeover and should pass a pause: its network calls are exempt, or the adopt call refuses at once naming the pause and `lym tender resume hub`.
The reconcile should wait for a short-lived weft push lock instead of stopping on it.

## From note drop-local-untracked-weft

## Seen (2026-11-11, Neve)

`lym adopt root --drop-local` archived the local tip of `push-notice-anchor-dir-weft` as `archive/push-notice-anchor-dir/5e8b95669e68`,
then failed to move the branch to origin's tip: `git reset --keep` refused because the untracked `_lym/reviews/code/generation-record.json` would be overwritten.
The file is one lym writes in the weft, so the operator has no clean way forward, and the adopt ended partial, holding back every other run under root.

## Idea

Once the tip is archived, drop-local moves the weft with a reset that also clears untracked lym files the target commit tracks, since the archive tag keeps the old state.

## From note adopt-stale-record-stops

## Seen (2026-11-11, Neve)

After a partial `lym adopt root --drop-local`, every later `lym adopt root` printed the same reconcile stops: pairs already removed with `lym fabric remove` and a run record cleared by `lym cleanup --apply` still showed as `gone` and `needs_replay`, and root stayed held back.
The call followed the unfinished adopt record instead of judging afresh.
`lym adopt --discard`, then `lym adopt root`, went through at once.

## Idea

A new `lym adopt <subject>` call whose stops come from a finished, failed worker judges the hub again instead of replaying the old record,
or names `lym adopt --discard` as the way forward when the stops no longer hold.

