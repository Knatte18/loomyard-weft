# lym adopt hangs silently behind a hub-wide pause

`adopt-held-by-pause` · Note · bug, tender, fabric

## Seen (2026-11-11, Neve)

A hub-wide `lym tender pause` set before the operator moved to another machine was still live when they came back.
`lym adopt root` then hung: its worker's first network call logged "held by a hub-wide pause" and waited with no word to the operator, whose terminal showed nothing.
Lifting the hub entry alone (`lym tender resume hub`) let it go on; the deferred records push that lift spawned then held the weft push lock,
and the adopt's origin reconcile stopped on `main-weft (busy)` after it had already set aside two pairs.

## Idea

An adopt is the operator's own takeover and should pass a pause: its network calls are exempt, or the adopt call refuses at once naming the pause and `lym tender resume hub`.
The reconcile should wait for a short-lived weft push lock instead of stopping on it.
