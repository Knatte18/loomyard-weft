# A landing notice is not lost when set late, when dobby was down, or when the orch is absent

`landing-watch-limits` · Note · enhancement, dobby, orch

## Seen (orch, 2026-11-01)

`landing-watch` (commit `c9d52bdb`) has dobby send the orch a line when a task with `notify_on_land` set lands.
Its commit lists known limits that no note covered:
- A `notify_on_land` set after the run passed Land gets no line.
- A run that passes Land while dobby is down long enough for a fresh state record to be written gets no line.
- The line needs a live orch; after three failed posts it is dropped.

The commit also records that the notice was never run end to end against a live orch.

## Open

- Whether any of these matter in practice, and which to close (operator to look at them).
- Setting the field on a run already past Land could answer at once instead of staying silent.
- A dropped line could wait in creel for the orch rather than be lost, which fits the dormant orch in `dormant-orch`.
