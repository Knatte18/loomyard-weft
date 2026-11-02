# A wake from hibernate leaves the hub-wide pause live

`hibernate-wake-leaves-pause` · Task · bug, hibernate, creel · running · Drift-Verify

## Seen (orch, 2026-11-02)

Neve hibernated through `lym hibernate request` on 2026-11-01 at 21:03 and woke on 2026-11-02 at 19:02.
After the wake the request read phase `done`, yet `_bolt/.lym/pause/record.json` still held the hub-wide `hibernate` pause as live.
`lym hibernate cancel` did nothing, since the request was done, and `lym adopt root` sat with every network call held ("held by a hub-wide pause since 2026-11-01T21:03:00"), which looked like a hang to the operator.
The watcher's wake notice failed twice (`_bolt/.lym/hibernate/watch.log`): first the receiver `ly:orch` does not exist, then the notice line was 105 KB, over creel's 8 KB limit for one hook delivery.

## Workaround

The orch ran `lym tender resume`, which lifted the pause, and adopt went on.

## Expected

- The wake lifts the pause it added before anything else, and a failed notice never keeps it.
- The wake notice reaches the orch by its real creel name and stays a short line, with any long detail in a file it names.
- `lym hibernate cancel` on a done request still lifts a pause the hibernate left live.
