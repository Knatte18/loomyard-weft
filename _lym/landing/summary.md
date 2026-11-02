# Lift the hibernate pause on every wake path and keep notices short

A wake from `lym hibernate request` could leave the hub-wide `hibernate` pause live while the request read `done`.
Every network call then sat held, `lym hibernate cancel` did nothing for a done request, and the operator had to run `lym tender resume`.
The wake notice also failed twice in the watcher: the receiver was refused, and the line was far over creel's limit for one hook delivery.

This change makes the wake always lift its own pause, keeps the notice to one short line, and lets `cancel` lift a pause a finished request left behind.

## What changes

- After the wake, the daemon in `hibernateengine` lifts its owned pause entry on every path.
  A failed write of the resuming or done phase, a reconcile that stops, and a notice that is never delivered no longer keep it.
  Only a lift error leaves the entry, and that fails the request.
- A reconcile stop on a branch every pair shares no longer keeps the hub entry live.
  The stopped runs get pair pause entries as for any other stop, the hub entry is lifted, and the notice names the stop before the runs.
- When the request recorded no owned entry, the entry kept from the pause stands in for it, so the reconcile still recognises the daemon's own hub entry.
- Every notice stays one short line.
  A line over one kilobyte, or one spanning lines, is written whole to `notice.txt` in the hibernate directory (`Deps.NoticePath`), and a short headline naming that file is queued instead.
  If the file cannot be written, the headline says so and the whole notice goes to the daemon's log.
  The failure notice takes the same path.
- A refused notice is queued again after a wait that starts at one second and doubles to thirty, until the waits add up to ten minutes, since the orch's session may not be listed right after a wake.
  The last refusal is returned, and the entry stays lifted either way.
- `lym hibernate cancel` on a request that has ended still lifts the pause entry it owns, by identity, and cancels nothing.
  A live daemon is never killed for an ended request, and the request file is left as it was.
- When the hub records no orch strand, the notice is not queued and the hub's `Notify` wiring logs a Warn naming the dropped notice.

## Design decisions

- The lift runs before the notice, so a notice failure can never keep the pause.
- Hub-wide pause entries are lifted by identity, as before, so an entry already resumed lifts nothing.
- The `cancel` change reuses the existing lift-by-identity path rather than adding one.

## To check by hand

- `notice-receiver-resolution-unchanged`: receiver resolution through `orchcli.NotifyPrime` is unchanged.
  The receiver refusal seen in `watch.log` is handled by the retry window and the Warn, not by a different name.
  Check that the orch's real creel name is what `NotifyPrime` addresses on a live hub.
- `notice-file-branches-untested`: the notice-file branches have unit tests in `internal/hibernateengine/watch_test.go`.
  The real-filesystem path under a live hub's hibernate directory and the 8 KB creel limit have not been run end to end.
- Wake a real machine from a `lym hibernate request` and confirm that `_bolt/.lym/pause/record.json` holds no live `hibernate` entry, and that the notice reaches the orch.
